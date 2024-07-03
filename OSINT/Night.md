# Night
Category: OSINT

Points: 121

Solves: 466

### Solution

![Night challenge photo](/images/NightChal.jpg)

The first step in many of these city challenges is to identify unique buildings in the background. Here, we see a very interesting tower with a tall antenna and a nearby building with a giant glass dome.

Using Google reverse image search, we can focus on the dome and antenna to find the unique buildings they belong to.

![Night reverse image search](/images/NightReverseImageSearch.png)

It turns out that the antenna is on Boston's Prudential Tower. The glass dome seen in the matched pictures further confirms our location. Now, we can put this in Google Maps and search around the area.

![Google Maps Investigation](/images/NightGoogleMapsExploration.png)

Looking at the original image, we can see train tracks on the left. This limits us to the tracks highlighted in red above.

Next, we know that we are driving towards the tower with tracks on the left-hand side of the road. This limits us to the road highlighted in orange.

From here, we could just use Street View to find our location since there isn't much road to cover. However, we can pinpoint our location even more using the on-ramp on the right side of the road. There is only one on-ramp (highlighted in yellow) that perfectly lines up with the overpass in front of the picture (highlighted in green).

This means we must be on the overpass highlighted in blue, as we have a high vantage point above the main road.

![Google Maps Street View](/images/NightGoogleMapsStreetView.png)

After going on street view and correctly aligning the camera, we can see that this matches the given photo.

Following the flag format `uiuctf{street name, city name}` yields the flag `uiuctf{Arlington Street, Boston}`





### Flag

```uiuctf{Arlington Street, Boston}```