# Chunky Boi
Category: OSINT

Points: 319

Solves: 225

### Solution

![Chunky Boi challenge photo](/images/ChunkyBoiChal.jpg)

We are asked to find the airplane model and location, so let's first start with the easy part, the model. Such a unique plane lends itself nicely to being reverse image searched.

![Chunky Boi reverse image search](/images/ChunkyBoiReverseImageSearch.png)

We now know the plane is a [Boeing C-17 Globemaster III](https://en.wikipedia.org/wiki/Boeing_C-17_Globemaster_III), but where could the airport be?

The overall atmosphere of the place, with evergreen trees and a mountainous background, suggests that this airport is in the North-West. The Alaska Airline planes also add to this general location.

Zooming in super close in the background, we can see a building named "Prologis":
![Chunky Boi Prologis](/images/ChunkyBoiPrologis.png)
After a bit of digging, we found: [Prologis SEA Cargo Center North](https://www.prologis.com/industrial-properties/building/tar00030-prologis-sea-cargo-center-north-922 
), indicating that we are in SEA-TAC.
![Chunky Boi Prologis Website](/images/ChunkyBoiPrologisWebsite.png)

Just to be safe we matched a few more buildings in the background.

![Background Buildings](/images/ChunkyBoiGoogleEarthBuildingE.png)
![Background Buildings](/images/ChunkyBoiChalBuildingE.png)
![Background Buildings](/images/ChunkyBoiGoogleEarthOtherBuilding.png)
![Background Buildings](/images/ChunkyBoiChalOtherBuilding.png)

Perfect, we now know we are in the right place, but SEA-TAC is huge, and we need coordinates with 3 decimal places of accuracy. At this point, you could just brute force coordinates by submitting all coordinates surrounding the middle of the airport, but where's the fun in that?

![Challenge Image Marked](/images/ChunkyBoiChalMarked.jpg)
![Google Maps Marked](/images/ChunkyBoiGoogleMapsMarked.png)

After pinpointing the plane, we get the coordinates approximately at `47.462320, -122.303737`. Truncating to 3 decimal places yields `47.462, -122.303`.
Combining this with the plane, `Boeing C-17 Globemaster III`, gives us the flag:
```uiuctf{Boeing C-17 Globemaster III, 47.462, -122.303}```

### False Starts, Dead Ends, and Other Exploration

**Meta Data**

Attempting to find location information in the metadata:

![Meta Data](/images/ChunkyBoiMetaData.png)

Unfortunately, there is no coordinate data, and the location information given (Guam, +2:00, -7:00) is incorrect.

**Overpass Turbo**

After noticing that the building in the background belonged to Prologis, I thought I could use Overpass Turbo to find Prologis buildings next to airports.

![Map of Prologis next to airport](/images/ChunkyBoiOverpassTurboQuery.png)

```json
[out:json][timeout:25];

// Define the USA area
area["name"="United States"]["boundary"="administrative"]["admin_level"=2]->.searchArea;

// Fetch Prologis properties in the US
(
  node(area.searchArea)["name"="Prologis"];
  way(area.searchArea)["name"="Prologis"];
  relation(area.searchArea)["name"="Prologis"];
)->.prologis;

// Find airports near the fetched Prologis properties
(
  node(around.prologis:5000)["aeroway"="aerodrome"];
  way(around.prologis:5000)["aeroway"="aerodrome"];
  relation(around.prologis:5000)["aeroway"="aerodrome"];
);

// Print results
out body;
>;
out skel qt;
```

Unfortunately, after searching through all these airports, none were correct. After solving the challenge, I noticed that the Prologis building wasn't labeled on Google Maps either.

**Tracking the Plane**

On the vertical stabilizer of the wing is a fleet number that we could track.

![Fleet Number](/images/ChunkyBoiFleetNumber.png)

Although we were able to [identify the plane](https://www.planespotters.net/airframe/boeing-c-17a-globemaster-iii-07-7182-united-states-air-force/e24v2d), none of the previously spotted locations were of use.


### Flag

```uiuctf{Boeing C-17 Globemaster III, 47.462, -122.303}```