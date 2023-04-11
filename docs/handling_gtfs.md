# Handling GTFS for retrieving Swedish regional transportation trafic historical data with python.

The KoDa API provides two main historical datasets: historical static and histrorical real-time

## Historical static

historical static is retrieved by a get request to the koda API, where fields *operator* and *date* are passed.

a .zip file is returned, it is a collection of .txt files following the GTFS Static format (Google) as described here:

>*"A GTFS feed is composed of a series of text files collected in a ZIP file. Each file models a particular aspect of transit information: stops, routes, trips, and other schedule data. "* — https://developers.google.com/transit/gtfs

One does not need to extract the all the .txt files from the zip file, the file can be read using python's module **gtfs_functions**.

```python
!pip install gtfs_functions

import gtfs_functions as gtfs
routes, stops, stop_times, trips, shapes = gtfs.import_gtfs(static-gtfs-filepath)
```

## Historical real-time

historical real time is retrieved by a get request to the koda API, where fields *operator*, *feed* and *date* are passed.

The main difference between querying historical rt and historical static from KoDa is that it can take time (1 till 60 mn):
>*"7-zip archives are not created automatically, but are instead created on-demand. The file will be created upon the first request. This request, and all other requests for the file will it’s being created, will receive an HTTP 202 Accepted response. If you receive this response, you can poll the URL every 30 seconds until you receive an HTTP 200 response along with the actual data. If someone else already requested the file earlier, you might be able to download the file immediately."* - https://www.trafiklab.se/api/trafiklab-apis/koda/historical-data/

Then, GTFS real time is formated way differently from GTFS static.

The zip file returned from the KoDa api has been compressed with 7-zip, built in extractors won't work so the py7zr module needs to be installed in the working environment

https://py7zr.readthedocs.io/en/latest/user_guide.html#install

```python
import py7zr
with py7zr.SevenZipFile(real-time-gtfs-filepath), 'r') as archive:
archive.extractall(path=savepath)
```

Here, no more *.txt*, but *.pb* files are stored in the desired folder after extraction. pb stands for protobuf or protocol buffer.

>*"Protocol buffers is a language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML or JSON, but smaller and faster. [...] Protobuf is the standard for GTFS-RT data. Since all producers use the same protobuf format and scheme, consumers only have to code an application once, whereafter it is available"* — https://www.trafiklab.se/docs/using-trafiklab-data/using-gtfs-files/the-protobuf-file-format/

Protocol buffers require a schema, for GTFS it is provided directly by google. Download the file to an appropriate folder.

https://developers.google.com/transit/gtfs-realtime/gtfs-realtime-proto

The protocol can be translated to python, in order to do so, download the latest Protocol Buffers for your system:
https://github.com/protocolbuffers/protobuf/releases/

then run the following command in a terminal, working from where the file is stored:

    protoc --python_out=. *.proto

The file ***gtfs_realtime_pb2.py*** will be outputed and will allow us to keep on going with python.

```python
import gtfs_realtime_pb2 #The script above
from protobuf_to_dict import protobuf_to_dict

feed = gtfs_realtime_pb2.FeedMessage()

with open('real-time-GTFS-pb-file', 'rb') as f:  #Important with rb (read binary)
    feed.ParseFromString(f.read())
    tripUpdates = protobuf_to_dict(feed)
```

At this point it is possible to associate the pb file to the static data through the fields *trip_id* and *stop_id*.
