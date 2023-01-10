# GIKI-Maps

import sys
from collections import defaultdict
import folium
import pandas as pd
import webbrowser
import csv
from geopy.distance import distance
import numpy as np
import point

Locations = pd.read_csv(
    r'D:\dsa_project\GIKIbuildingLocations.csv')
# view the dataset
print(Locations.head())
center = [34.069482899664195, 72.64248285607844]
map_GIKI = folium.Map(location=center, zoom_start=17)
for index, loc in Locations.iterrows():
    location = [loc['latitude'], loc['longitude']]
    folium.Marker(location, popup=f'Name:{loc["faculty"]}\n ').add_to(map_GIKI)

# paths = pd.read_csv(r'D:\dsa_project\GIKIPATHS.csv')
# print("paths:", paths.to_numpy())

file = open("GIKIPATHS.csv", newline='')
# result_list = list(csv.reader(file))
# print(result_list)
# result_2D=np.array(result_list)

# forming paths for dictionary

paths = csv.DictReader(file)
# for row in paths:
#     print(row['loc1lon'])

# result_2D = result_2D[1::]
# print(result_2D)
# paths = list(paths)

list_of_paths=list(list(paths))

print(list_of_paths)
# for row in paths:
#     print(row)
# open the file in the write mode
#f = open(r"C:\Users\Lenovo\OneDrive\Desktop\DSA PROJECT\distance.csv", "w")

# A Python program for Dijkstra's shortest
# path algorithm for adjacency
# list representation of graph


class Heap():

    # By using the “self” we can access the attributes and methods of the class in python.

    def __init__(self):
        self.array = []
        self.size = 0
        self.pos = []

    def newMinHeapNode(self, v, dist):
        minHeapNode = [v, dist]
        return minHeapNode

    # A utility function to swap two nodes
    # of min heap. Needed for min heapify
    def swapMinHeapNode(self, a, b):
        t = self.array[a]
        self.array[a] = self.array[b]
        self.array[b] = t

    # A standard function to heapify at given idx
    # This function also updates position of nodes
    # when they are swapped.Position is needed
    # for decreaseKey()
    def minHeapify(self, idx):
        smallest = idx
        left = 2*idx + 1
    # used to access the left child
        right = 2*idx + 2
    # access the right child
        if (left < self.size and self.array[left][1] < self.array[smallest][1]):
            smallest = left

        if (right < self.size and self.array[right][1] < self.array[smallest][1]):
            smallest = right

        # The nodes to be swapped in min
        # heap if idx is not smallest
        if smallest != idx:

            # Swap positions
            self.pos[self.array[smallest][0]] = idx
            self.pos[self.array[idx][0]] = smallest

            # Swap nodes
            self.swapMinHeapNode(smallest, idx)

            self.minHeapify(smallest)

    # Standard function to extract minimum
    # node from heap
    def extractMin(self):

        # Return NULL wif heap is empty
        if self.isEmpty() == True:
            return

        # Store the root node
        root = self.array[0]

        # Replace root node with last node
        lastNode = self.array[self.size - 1]
        self.array[0] = lastNode

        # Update position of last node
        self.pos[lastNode[0]] = 0
        self.pos[root[0]] = self.size - 1

        # Reduce heap size and heapify root
        self.size -= 1
        self.minHeapify(0)

        return root

    def isEmpty(self):
        return True if self.size == 0 else False

    def decreaseKey(self, v, dist):
        # decreaseKey inorder to go back to the parent node as this isn't the shortest root to take

        # Get the index of v in heap array

        i = self.pos[v]

        # Get the node and update its dist value
        self.array[i][1] = dist

        # Travel up while the complete tree is
        # not heapified. This is a O(Logn) loop
        while (i > 0 and self.array[i][1] < self.array[(i - 1) // 2][1]):
            #  primarily used when you require an integer or need to return the smallest integer less than or equal to the input.

            # Swap this node with its parent
            self.pos[self.array[i][0]] = (i-1)//2
            self.pos[self.array[(i-1)//2][0]] = i
            self.swapMinHeapNode(i, (i - 1)//2)

            # move to parent index
            i = (i - 1) // 2

    # A utility function to check if a given
    # vertex 'v' is in min heap or not
    def isInMinHeap(self, v):
        if self.pos[v] < self.size:
            return True
        return False


def printArr(dist, n):
    print("Vertex\tDistance from source")
    for i in range(n):
        # print("%d\t\t%d" % (i, dist[i]))
        print(f"{i}\t\t{dist[i]}")
        # folium.PolyLine([path1, path2], weight=4).add_to(map_GIKI)

        # i is the current vertex number
        # the weight is stored inside dist[i] array


class Graph():

    def __init__(self, V):
        self.V = V
        self.graph = defaultdict(list)

    # Adds an edge to an undirected graph
    def addEdge(self, src, dest, weight):

        # Add an edge from src to dest. A new node
        # is added to the adjacency list of src. The
        # node is added at the beginning. The first
        # element of the node has the destination
        # and the second elements has the weight
        newNode = [dest, weight]
        self.graph[src].insert(0, newNode)

        # Since graph is undirected, add an edge
        # from dest to src also
        newNode = [src, weight]
        self.graph[dest].insert(0, newNode)

    # The main function that calculates distances
    # of shortest paths from src to all vertices.
    # It is a O(ELogV) function
    def dijkstra(self, src):

        V = self.V  # Get the number of vertices in graph
        dist = []  # dist values used to pick minimum
        # weight edge in cut

        # minHeap represents set E
        minHeap = Heap()

        # Initialize min heap with all vertices.
        # dist value of all vertices
        for v in range(V):
            dist.append(1e7)
            minHeap.array.append(minHeap.newMinHeapNode(v, dist[v]))
            minHeap.pos.append(v)

        # Make dist value of src vertex as 0 so
        # that it is extracted first
        minHeap.pos[src] = src
        dist[src] = 0
        minHeap.decreaseKey(src, dist[src])

        # Initially size of min heap is equal to V
        minHeap.size = V

        # In the following loop,
        # min heap contains all nodes
        # whose shortest distance is not yet finalized.
        while minHeap.isEmpty() == False:

            # Extract the vertex
            # with minimum distance value
            newHeapNode = minHeap.extractMin()
            u = newHeapNode[0]

            # Traverse through all adjacent vertices of
            # u (the extracted vertex) and update their
            # distance values
            for pCrawl in self.graph[u]:
                v = pCrawl[0]
                # print(f"v: {v}")

                # If shortest distance to v is not finalized
                # yet, and distance to v through u is less
                # than its previously calculated distance
        # 1e7 represents 10^7 scientific notation
                if (minHeap.isInMinHeap(v) and dist[u] != 1e7 and pCrawl[1] + dist[u] < dist[v]):
                    dist[v] = pCrawl[1] + dist[u]

                    # update distance value
                    # in min heap also
                    minHeap.decreaseKey(v, dist[v])
        return [dist, V]
        # printArr(dist, V)



# create the csv writer(249)
# iterrows allows to iterate over each row in the data frame
graph = Graph(12)
#writer = csv.writer(f)

for pth in paths:
    path1 = [pth['loc1lon'], pth['loc1lat']]
    path2 = [pth['loc2lon'], pth['loc2lat']]
    # path1 = [pth[1], pth[2]]
    # path2 = [pth[4], pth[5]]
    #km = distance(path1, path2)
    # writer.writerow(km)
    print(pth['vertex'])
    v1=pth['vertex']
    v1 = int(v1)
    v2=pth['vertex2']
    v2 = int(v2)
    d=pth['distance']
    d= float(d)
    print('this is path1')
    print(path1)
    print('this is path 2')
    print(path2)
    graph.addEdge(v1, v2, d)
    
    folium.PolyLine(paths, weight=4).add_to(map_GIKI)

# vertex = 3
# print("Applying dijkstra's algorithm on vertex {}".format(vertex))
# print(graph.dijkstra(vertex))
# save map to html file

map_GIKI.save('index.html')
webbrowser.open("index.html")
