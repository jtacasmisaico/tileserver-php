TileServer PHP: MapTiler and MBTiles maps via WMTS
==================================================

This server distributes maps to desktop, web, and mobile applications from
a standard Apache+PHP web hosting.

Try a live demo at: http://tileserver.maptiler.com/

It is a free and open-source project implementing OGC WMTS standard for
pre-rendered map tiles made with [MapTiler](http://www.maptiler.com/), GDAL2Tiles,
or available as MBTiles files.

It is the easiest and cheapest way how to serve zoomable maps in a
standardized way - practically from any ordinary web hosting.

It is easy to install - just copy the project files to a PHP-enabled
directory along with your map data.

It comes with an online interface showing the list of the maps and step-by-step guides for online mapping libraries (Google Maps API, Leaflet, OpenLayers, OL3, MapBox JS, ArcGIS JS) and various desktop GIS software:

![tileserver-screenshot](https://f.cloud.github.com/assets/59284/1041807/a040160c-0fdb-11e3-8941-ab367b2a648d.png)

This project is developed in PHP, not because it is the best language for
development of web applications, but because it maximally simplify the
deployment on large number of web hosting providers including various free
web hostings.

Tiles are served directly by Apache with mod_rewrite rules as static files
and therefore are very fast and with correct HTTP caching headers.
Only XML metadata are delivered via PHP.
MBTiles are served via PHP, and are therfore slower, unless they are unpacked with mbutil.

[MapTiler](http://www.maptiler.com/) can render GeoTIFF, ECW, MrSID, GeoPDF into compatible map tiles. JPEG, PNG, GIF and TIFF with scanned maps or images without geolocation can be turned into standard map layers with the visual georeferencing functionality (http://youtu.be/eJxdCe9CNYg).

[![MapTiler - mapping tiles](https://cloud.githubusercontent.com/assets/59284/3037911/583d7810-e0c6-11e3-877c-6a7747b80dd3.jpg)](http://www.maptiler.com/)

Requirements:
-------------

- Apache webserver (with mod_rewrite / .htaccess supported)
- PHP 5.2+ with SQLite module (php5-sqlite)

(or another webserver implementing mod_rewrite rules and PHP)

Installation:
-------------

Download the project files as a [zip archive](https://github.com/klokantech/tileserver-php/archive/master.zip) or source code from GitHub and unpack it into a web-hosting of your choice.

If you access the web address relevant to the installation directory, 
the TileServer.php Server should display you a welcome message and further
instructions.

Then you can upload to the web hosting your mapping data - a directory with
tiles rendered with [MapTiler](http://www.maptiler.com/).

Tiles produced by open-source GDAL2Tiles or MapTiler and tiles in .mbtiles
files can be easily converted to required structure (XYZ with top-left origin
and metadata.json file). The open-source utility [mbutil](https://github.com/mapbox/mbutil) produces
exactly the required format.

Direct reading of .mbtiles files is supported, but with decreased performance
compared to the static files in a directory. The advantage is easier data management,
especially upload over FTP or similar protocols.

Supported protocols:
--------------------

- OpenGIS WMTS 1.0.0
  
  The Open Geospatial Consortium (OGC) Web Map Tile Service (WMTS)
  Both KVP and RESTful version 1.0.0:
  http://www.opengeospatial.org/standards/wmts/
  
  Target is maximal compliance to the standard.
  
  Exposed at http://[...]/wmts
  
- OSGeo TMS 1.0.0

  The OSGeo Tile Maps Service, but with inverted y-coordinates:
  http://wiki.osgeo.org/wiki/Tile_Map_Service_Specification
  This means request compatible with OpenStreetMap tile servers.

  Target is "InvertedTMS" implementation used by the ArcBruTile client
  which is available from http://arcbrutile.codeplex.com/ and uses
  flipped y-axis.

  Exposed at http://[...]/tms
  
- TileJSON.js

  Metadata about the individual maps in a ready to use form for web
  clients following the standard http://mapbox.com/developers/tilejson/
  and with support for JSONP access.

  Exposed at http://[...]/layer.jsonp
  
- Direct access with XYZ tile requests (to existing tiles in a directory
  or to .mbtiles)

  Compatible with Google Maps API / Bing SDK / OpenStreetMap clients.
  
  Exposed at http://[...]/layer/z/x/y.ext
  
- MapBox UTFgrid request (for existing tiles in .mbtiles with UTFgrid support). Callback is supported 

  Example https://www.mapbox.com/demo/visiblemap/
  Specification https://github.com/mapbox/utfgrid-spec
  
  Exposed at http://[...]/layer/z/x/y.grid.json
  

To use the OGC WMTS standard point your client (desktop or web) to the URL
of 'directory' where you installed tileserver.php project with suffix "wmts".
For example: http://www.example.com/directory/wmts

If you have installed the project into a root directory of a domain, then the address is: http://www.example.com/wmts

The supported WMTS requests includes:

GetCapabilities RESTful/KVP:
 
   http://[...]/1.0.0/WMTSCapabilities.xml
   http://[...]?service=wmts&request=getcapabilities&version=1.0.0
 
GetTile RESTful/KVP:
 
   http://[...]/layer/[ANYTHING-OPTIONAL][z]/[x]/[y].[ext]
   http://[...]?service=wmts&request=getTile&layer=[layer]&tilematrix=[z]&tilerow=[y]&tilecol=[y]&format=[ext]
   
Other example requests are mentioned in the .htaccess.

Performance from the web clients
--------------------------------

It is highly recommended to map several domain names to the service, such as:
http://a.example.com/, http://b.example.com/, http://c.example.com/.
This can be done with DNS CNAME records pointing to your hosting.
The reason for this is that traditionally browsers will not send more then two
simultaneous http request to the same domain - with multiple domains for the
same server you can better saturate the network and receive the maps faster.

Performance
-----------

In case the data are available in a form of directory with XYZ tiles, then
Apache webserver is serving these files directly as WMTS RESTful or KVP.

This means performance is excellent, maps are delivered very fast and large
number of concurrent visitors can be handled even with quite a low-end
hardware or cheap/free web hosting providers.

Mod_rewrite rules are utilized to ensure the HTTP requests defined in the OCG
WMTS standard are served, and Apache preserve standard caching headers & eTag.

The performance should be significantly better then performance of any other
tile caching project (such as TileCache.org or GeoWebCache).

Performance graph for "apache static" comparing other tile caching projects
is available online at:
http://code.google.com/p/mod-geocache/wiki/PreliminaryBenchmark

Limits of actual implementation
-------------------------------

With intention, in this moment the project supports only:
- Mercator tiles (a la OpenStreetMap) and Geodetic tiles (WGS84 unprojected)
  with known and described tiling scheme.
- All tiles must be 256x256 pixels.
- We enforce and require XYZ (top-left origin) tiling schema (even for TMS).

Password protection
-------------------

HTTP Simple Authentication can be easily added to the server.
Edit the .htaccess and add these lines:

    AuthUserFile /full/path/to/.htpasswd
    AuthType Basic
    AuthName "Secure WMTS"
    Require valid-user

Create a file called .htpasswd with user:password format.
You can use a command-line utility:

$ htpasswd -c .htpasswd [your-user-login]

Or an online service:

http://www.htaccesstools.com/htpasswd-generator/

HTTPS / SSL support
-------------------

TileServer.php can run without any problems over HTTPS, if required.

Microsoft Windows web-hosting
-----------------------------

The TileServer.php should run on Windows-powered webservers with Apache
installation if PHP 5.2+ and mod_rewrite are available.

With the IIS webserver hosting, you may need PHP and IIRF module
(http://iirf.codeplex.com/) and alter appropriately the rewrite rules.

Credits / Contributors
----------------------

Project developed initially by Klokan Technologies GmbH, Switzerland in
cooperation with National Oceanic and Atmospheric Administration - NOAA, USA.

- Petr Pridal - Klokan Technologies GmbH <petr.pridal@klokantech.com>
- Jason Woolard - NOAA <jason.woolard@noaa.gov>
- Jon Sellars - NOAA <jon.sellars@noaa.gov>
- Dalibor Janak - Klokan Technologies GmbH <dalibor.janak@klokantech.com>

Tested WMTS/TMS clients
-----------------------

- QuantumGIS Desktop 1.9+ - open with Layer->Add WMS layer
  http://www.qgis.org/
- ESRI ArcGIS Desktop 10.1+ - native WMTS implementation supported
  http://www.esri.com/software/arcgis/arcgis-for-desktop
- ESRI ArcGIS Online - loading via WMTS protocol
  http://www.arcgis.com/
- ArcBruTiles plugin for ArcGIS 9.3+ - via TMS endpoint
  http://arcbrutile.codeplex.com/
- OpenLayers WMTS Layer - including parsing GetCapabilities
  http://www.openlayers.org/
- GAIA - native WMTS (issues with 3857 to be fixed)
  http://www.thecarbonproject.com/gaia.php
- MapBox.js - the loading of maps via TileJSON, interaction layer supported
  https://www.mapbox.com/mapbox.js

BSD License
-----------

Copyright (C) 2015 Klokan Technologies GmbH (http://www.klokantech.com/)
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met: 

1. Redistributions of source code must retain the above copyright notice, this
   list of conditions and the following disclaimer. 
2. Redistributions in binary form must reproduce the above copyright notice,
   this list of conditions and the following disclaimer in the documentation
   and/or other materials provided with the distribution. 

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR
ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
