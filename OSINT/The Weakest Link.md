# The Weakest Link
Category: OSINT
Points: 393
Solves: 149

### Solution
```LISA and the secret business partner have a secret Spotify collaboration planned together. Unfortunately, neither of them have the opsec to keep it private. See if you can figure out what it is!```

After solving [Hip With the Youth](https://github.com/EnchLolz/UIUCTF-2024/blob/main/OSINT/Hip%20WIth%20the%20Youth.md) and [An Unlikely Partnership](https://github.com/EnchLolz/UIUCTF-2024/blob/main/OSINT/An%20Unlikely%20Partnership.md), we know that the secret business partner is UIUC Chan, and we should now find her Spotify account.

#### Option 1
To do this, we can just go on the Spotify app and search for UIUC Chan.

![UIUC Chan Spotify search](/images/TheWeakestLink_UIUC-Chan_search.png)

#### Option 2
I later found that her Spotify account was linked on her LinkedIn contact info.

![UIUC Chan LinkedIn contact info](/images/TheWeakestLink_UIUC-Chan_LinkedIn_contact.png)



![UIUC Chan Spotify Profile](/images/TheWeakestLink_UIUC-CHAN_Spotify_profile.png)

Now that we are on her profile we can start looking through her public info. Furthermore, `I love music! I quite literally play it at all times. Good thing none of my secret Spotify collab projects are on my profile, that would be so embarrassing!` on her LinkedIn bio strongly suggests that she made a collaborative playlist with LISA.

Where could we find this playlist thought? After a while of random searching, clicking on Friend Activity reveals that UIUC Chan is listening to a playlist `songs for train lovers`.
\* note that you must follow UIUC Chan for her to show up on friends list 
\* also the friend activity button doesn't exist on the browser version of spotify, I am using the desktop app for this writeup

![Friend Activity](/images/TheWeakestLink_friend_activity.png)

Clicking into this playlist shows the flag.

![Playlist with flag](/images/TheWeakestLink_Flag.png)


### False Starts, Dead Ends, and Other Exploration

**Spotify Stuff**

Honestly, there was a lot of trial an error in this process since there were so many things to look through. Scrolling though the public playlist `songs to hail to the orange and hail to the blue to` we get to listen to 3 banger songs, but none of them contain the flag.

![Not flag playlist](/images/TheWeakestLink_UIUC-Chan_playlist.png)

I tried adding songs and saving the playlist to see if I would get access to more information, but after hours of random clicking nothing worked. I even put the pfp of UIUC Chan and the Album Cover through aperisolve to see if there was any stego.

**Spotify API**

I then thought, maybe I could use the [Spotify API](https://developer.spotify.com/documentation/web-api) to gain info about accounts. 

```bash
>>> curl --request GET \
    --url https://api.spotify.com/v1/users/31d2lcivqdieyl4qzx25vfmp6jt4 \
    --header 'Authorization: Bearer TOKEN'
>>> curl --request GET \
    --url https://api.spotify.com/v1/playlists/21DFySESIGEwhj1xSednEW \
    --header 'Authorization: Bearer TOKEN' > PlaylistInfo
```

Going through user information and playlist information unfortunately didn't yield any results that weren't already on her profile.

**After Challenge Reflection**

I also thought about looking for a LISA account on Spotify but eventually gave up since it was too common of a username. I also tried to find LISA by looking through the followers of UIUC Chan but didn't find it. However, after solving this, there is a LISA account, and it follows UIUC Chan?! I guess I rushed too quickly through all the possibilities. Hopefully next time I can slow things down and take a more systematic approach. :pensive:


### Flag

```uiuctf{7rU1Y_50N65_0F_7H3_5UMM3r_432013}```