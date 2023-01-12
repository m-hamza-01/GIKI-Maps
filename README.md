from collections import defaultdict
import folium
import pandas as pd
import webbrowser
from geopy.distance import distance
from ursina import *
from PIL import Image
from ursina.prefabs.dropdown_menu import DropdownMenu, DropdownMenuButton


def main1(dd):

    Locations = pd.read_csv(
        r'C:\Users\Lenovo\OneDrive\Desktop\DSA PROJECT\GIKIbuildingLocations.csv')
    # view the dataset
    print(Locations.head())
    center = [34.069482899664195, 72.64248285607844]
    map_GIKI = folium.Map(location=center, zoom_start=17)
    for index, loc in Locations.iterrows():
        location = [loc['latitude'], loc['longitude']]
        folium.Marker(
            location, popup=f'Name:{loc["faculty"]}\n ').add_to(map_GIKI)

    paths = pd.read_csv(
        r'C:\Users\Lenovo\OneDrive\Desktop\DSA PROJECT\GIKIPATHS.csv')

    class Heap():
        def __init__(self):
            self.array = []
            self.size = 0
            self.pos = []

        def newMinHeapNode(self, v, dist):
            minHeapNode = [v, dist]
            return minHeapNode

        def swapMinHeapNode(self, a, b):
            t = self.array[a]
            self.array[a] = self.array[b]
            self.array[b] = t

        def minHeapify(self, idx):
            smallest = idx
            left = 2*idx + 1
            right = 2*idx + 2
            if (left < self.size and self.array[left][1] < self.array[smallest][1]):
                smallest = left

            if (right < self.size and self.array[right][1] < self.array[smallest][1]):
                smallest = right

            if smallest != idx:

                self.pos[self.array[smallest][0]] = idx
                self.pos[self.array[idx][0]] = smallest

                self.swapMinHeapNode(smallest, idx)

                self.minHeapify(smallest)

        def extractMin(self):

            if self.isEmpty() == True:
                return
            root = self.array[0]

            lastNode = self.array[self.size - 1]
            self.array[0] = lastNode

            self.pos[lastNode[0]] = 0
            self.pos[root[0]] = self.size - 1

            self.size -= 1
            self.minHeapify(0)

            return root

        def isEmpty(self):
            return True if self.size == 0 else False

        def decreaseKey(self, v, dist):
            i = self.pos[v]
            self.array[i][1] = dist
            while (i > 0 and self.array[i][1] < self.array[(i - 1) // 2][1]):
                self.pos[self.array[i][0]] = (i-1)//2
                self.pos[self.array[(i-1)//2][0]] = i
                self.swapMinHeapNode(i, (i - 1)//2)

                i = (i - 1) // 2

        def isInMinHeap(self, v):
            if self.pos[v] < self.size:
                return True
            return False

    def printArr(dist, n):
        print("Vertex\tDistance from source")
        for i in range(n):
            print(f"{i}\t\t{dist[i]}")
            # folium.PolyLine([path1, path2], weight=4).add_to(map_GIKI)

    class Graph():

        def __init__(self, V):
            self.V = V
            self.graph = defaultdict(list)

        def addEdge(self, src, dest, weight):
            newNode = [dest, weight]
            self.graph[src].insert(0, newNode)
            newNode = [src, weight]
            self.graph[dest].insert(0, newNode)

        def dijkstra(self, src):

            V = self.V
            dist = []
            minHeap = Heap()
            for v in range(V):
                dist.append(1e7)
                minHeap.array.append(minHeap.newMinHeapNode(v, dist[v]))
                minHeap.pos.append(v)

            minHeap.pos[src] = src
            dist[src] = 0
            minHeap.decreaseKey(src, dist[src])

            minHeap.size = V

            while minHeap.isEmpty() == False:

                newHeapNode = minHeap.extractMin()
                u = newHeapNode[0]

                for pCrawl in self.graph[u]:
                    v = pCrawl[0]
                    # print(f"v: {v}")

                    if (minHeap.isInMinHeap(v) and dist[u] != 1e7 and pCrawl[1] + dist[u] < dist[v]):
                        dist[v] = pCrawl[1] + dist[u]

                        minHeap.decreaseKey(v, dist[v])

            printArr(dist, V)

    # create the csv writer
    graph = Graph(48)
    #writer = csv.writer(f)

    for index, pth in paths.iterrows():
        path1 = [pth['loc1lon'], pth['loc1lat']]
        path2 = [pth['loc2lon'], pth['loc2lat']]
        #km = distance(path1, path2)
        # writer.writerow(km)
        # print(pth['vertex'])
        v1 = pth['vertex']
        v1 = int(v1)
        v2 = pth['vertex2']
        v2 = int(v2)
        d = pth['distance']
        d = float(d)

        folium.PolyLine([path1, path2], weight=4).add_to(map_GIKI)

        graph.addEdge(v1, v2, d)

    graph.dijkstra(0)
    # save map to html file

    map_GIKI.save('index.html')
    webbrowser.open("index.html")


app = Ursina()


bg = Entity(model='quad', texture=Texture("./map.png"), scale=(22,
            15), z=10, color=color.dark_gray)  # set a PIL texture
cube = Entity(model="cube", color=color.red,
              texture="white_cube", scale=2, position=(0, 1))


def update():
    cube.rotation_x = cube.rotation_x + 0.25
    cube.rotation_y = cube.rotation_y + 0.5


logo = Text(text="GIKIGO", font={'Calibri', 'bold'},
            scale=5, x=-0.25, y=0.45, color=color.red)
b1 = Button(text="PLAY", color=color.red, scale=(0.3, 0.1), position=(0, -0.3))


def loc():
    b1.enabled = False
    cube.enabled = False
    logo.enabled = False
    b2 = Button(text="Back", color=color.red,
                scale=(0.3, 0.1), position=(-0.6, -0.3))

    def b2_update():
        b1.enabled = True
        cube.enabled = True
        b2.enabled = False
        logo.enabled = True

        DropdownMenu.visible = False

    b2.on_click = b2_update

    DropdownMenu('Select your Location', position=(-0.2, 0.5), color=color.red, buttons=(
        DropdownMenuButton('Auditorium', on_click=main1(10)),
        DropdownMenuButton('Ayan Gardens', on_click=main1),
        DropdownMenuButton('Academic Block', on_click=main1),
        DropdownMenuButton('Brabers Building', on_click=main1),
        DropdownMenuButton('Central Library', on_click=main1),
        DropdownMenuButton('FES', on_click=main1),
        DropdownMenuButton('FCSE', on_click=main1),
        DropdownMenuButton('FMCE', on_click=main1),
        DropdownMenuButton('FME', on_click=main1),
        DropdownMenuButton('Faculty Club', on_click=main1),
        DropdownMenuButton('Giki Sports Complex', on_click=main1),
        DropdownMenuButton('Giki Tennis Court', on_click=main1),
        DropdownMenuButton('Giki Gate', on_click=main1),
        DropdownMenuButton('Giki Guest House', on_click=main1),
        DropdownMenuButton('Graduate Hostel', on_click=main1),
        DropdownMenuButton('Hostel 1', on_click=main1),
        DropdownMenuButton('Hostel 2', on_click=main1),
        DropdownMenuButton('Hostel 3', on_click=main1),
        DropdownMenuButton('Hostel 4', on_click=main1),
        DropdownMenuButton('Hostel 5', on_click=main1),
        DropdownMenuButton('Hostel 6', on_click=main1),
        DropdownMenuButton('Hostel 7(GH)', on_click=main1),
        DropdownMenuButton('Hostel 8', on_click=main1),
        DropdownMenuButton('Hostel 9', on_click=main1),
        DropdownMenuButton('Hostel 10', on_click=main1),
        DropdownMenuButton('Hostel 11', on_click=main1),
        DropdownMenuButton('Hostel 12', on_click=main1),
        DropdownMenuButton('Habib Bank', on_click=main1),
        DropdownMenuButton('Hot and Spicy', on_click=main1),
        DropdownMenuButton('Incubation Center', on_click=main1),
        DropdownMenuButton('Laundry', on_click=main1),
        DropdownMenuButton('Medical Center(MC)', on_click=main1),
        DropdownMenuButton('Parents Lodge', on_click=main1),
        DropdownMenuButton('Raju Hotel', on_click=main1),
        DropdownMenuButton('Residential Colony Masjid', on_click=main1),
        DropdownMenuButton('Service Center', on_click=main1),
        DropdownMenuButton('Tuck Mosque', on_click=main1),
        DropdownMenuButton('Togic', on_click=main1),
    )
    )

    DropdownMenu('Select your Destination', position=(0.2, 0.5), color=color.black, buttons=(
        DropdownMenuButton('Auditorium'),
        DropdownMenuButton('Ayan Gardens'),
        DropdownMenuButton('Academic Block'),
        DropdownMenuButton('Brabers Building'),
        DropdownMenuButton('Central Library'),
        DropdownMenuButton('FES'),
        DropdownMenuButton('FCSE'),
        DropdownMenuButton('FMCE'),
        DropdownMenuButton('FME'),
        DropdownMenuButton('Faculty Club'),
        DropdownMenuButton('Giki Sports Complex'),
        DropdownMenuButton('Giki Tennis Court'),
        DropdownMenuButton('Giki Gate'),
        DropdownMenuButton('Giki Guest House'),
        DropdownMenuButton('Graduate Hostel'),
        DropdownMenuButton('Hostel 1'),
        DropdownMenuButton('Hostel 2'),
        DropdownMenuButton('Hostel 3'),
        DropdownMenuButton('Hostel 4'),
        DropdownMenuButton('Hostel 5'),
        DropdownMenuButton('Hostel 6'),
        DropdownMenuButton('Hostel 7(GH)'),
        DropdownMenuButton('Hostel 8'),
        DropdownMenuButton('Hostel 9'),
        DropdownMenuButton('Hostel 10'),
        DropdownMenuButton('Hostel 11'),
        DropdownMenuButton('Hostel 12'),
        DropdownMenuButton('Habib Bank'),
        DropdownMenuButton('Hot and Spicy'),
        DropdownMenuButton('Incubation Center'),
        DropdownMenuButton('Laundry'),
        DropdownMenuButton('Medical Center(MC)'),
        DropdownMenuButton('Parents Lodge'),
        DropdownMenuButton('Raju Hotel'),
        DropdownMenuButton('Residential Colony Masjid'),
        DropdownMenuButton('Service Center'),
        DropdownMenuButton('Tuck Mosque'),
        DropdownMenuButton('Togic'),
    )
    )


b1.on_click = loc

app.run()
