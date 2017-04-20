<h1> Using unsupervised learning to calculate optimal Seattle PD station locations </h1>

<img src="images/pd_locations.png" width="800">
<h4> Figure: Seattle police responses shown in red. K-Means optimized SPD station locations shown in blue and actual station locations </h4> shown in green.
I decided to use some simple machine learning to calculate the optimal SPD station locations with data for around 1.5 million police responses. This was a very fun project because I had to ‘manually’ implement the K-means algorithm so I could use a custom distance measurement (see src folder). K-means is a method used to allow computers to mathematically recognize patterns and group data without being given explicit instructions.

Using the SPD stations example, the algorithm randomly places six stations around the map. Next, each police response is mapped to the nearest station so six groups of police responses are formed. The stations are then relocated to the center of their group and the responses are remapped to the nearest new station location. Repeat this 1000 times or so, and the algorithm can determine the optimal locations of SPD stations around the city. The calculations took a couple of hours because calculating distance using latitude and longitude is surprisingly non-trivial with curvature of the Earth and the fact that longitudinal distance varies with latitude.

I noticed a couple really interesting things here.
- The algorithm has no knowledge of the locations of the real SPD stations, yet 4 of the 6 locations calculated are quite close to the real locations. This is very surprising because I’d imagine the real locations were selected decades ago without much scientific evaluation.
- According to the model, the North end should have 3 stations instead of 1. Is this lack of stations due to the relatively late incorporation of the North end?

Fine print disclaimer – This is a quick and dirty after school project run on a laptop so there are some real limitations to the model. The model does not understand water, traffic, and street distances because I did not have the time or computing power to implement these things. These results should be taken with a grain of salt.


```python
import random
import numpy as np
from geopy.distance import vincenty
from collections import defaultdict


def k_means(X, k=5, max_iter=1000):
    """
    Inputs:
    X = 2 x m dataframe of latitude and longitude (float)
    k = number of centroids to use (int)
    max_iter = maximum number of iteratations (int)
    Return:
    centroids = dataframe latitude and longitude of centroids (float)
    clusters = dict mapping centoids to observations
    """

    # Zip data latitude and longitude into tuples for Vincenty distance
    lats = [lat for lat in X.Latitude]
    longs = [long_ for long_ in X.Longitude]
    X_list = zip(lats, longs)

    # Zip centroid latitude and longitude into tuples for Vincenty distance
    lats_cent = [lat for lat in random.sample(X.Latitude, k)]
    longs_cent = [long_ for long_ in random.sample(X.Longitude, k)]
    centroids = zip(lats_cent, longs_cent)

    for i in xrange(max_iter):
        clusters = defaultdict(list)
        # Assign each point to nearest cluster for nearest centroid
        for x in X_list:
            distances = [vincenty(x, centroid) for centroid in centroids]
            centroid = centroids[np.argmin(distances)]
            clusters[centroid].append(x)

        # Move centroids to center of their clusters
        new_centroids = []
        for centroid, pts in clusters.iteritems():
            new_centroid = np.mean(pts, axis=0)
            new_centroids.append(tuple(new_centroid))

        # Stop iterating if optimized else use new centroids for next iter
        if set(new_centroids) == set(centroids):
            break
        else:
            centroids = new_centroids

    return centroids, clusters
```

<img src="images/logos/seattle.png" width="120"> <img src="images/logos/python.png" width="120"> <img src="images/logos/atom.png" width="120"> <img src="images/logos/matplotlib.png" width="120"> <img src="images/logos/jupyter.png" width="120"> <img src="images/logos/linux.png" width="120">
