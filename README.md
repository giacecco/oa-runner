oa-runner
=========

oa-runner is a collection of scripts to support runners who want to contribute to [Open Addresses UK](http://openaddressesuk.org) by calculating opportunities for investigating addresses that are nearby their planned running courses.

**NOTE: this is a working prototype only**, but you are very welcome to feedback and contribute. Please use this repository's [issues section](https://github.com/Digital-Contraptions-Imaginarium/oa-runner/issues).

##Setup
- Setup any recent Java runtime environment suitable to your system, so that it can run from the command line using the *java* command. This is required to run the *FitCSVTool* tool, part of the FIT SDK. We did our testing using Apple MacOS' Java SE runtime environment version 1.7.0_21.
- Install the [PhantomJS](http://phantomjs.org/) command line utility. On MacOS, using [Homebrew](http://brew.sh/), it's as simple as:

	```
	brew install phantomjs
	```

- If you want to use *.fit* files, download the FIT SDK from [http://www.thisisant.com/resources/fit](http://www.thisisant.com/resources/fit) and uncompress it in *etc* (e.g. *etc/FitSDKRelease13.10*).
- Download the latest CSV edition of Office for National Statistics' "Postcode Directory" Open Data dataset from [https://geoportal.statistics.gov.uk/geoportal/catalog/content/filelist.page?redirect=Docs/PostCodes/](https://geoportal.statistics.gov.uk/geoportal/catalog/content/filelist.page?redirect=Docs/PostCodes/) and uncompress it in *data* (e.g. *data/ONSPD_NOV_2014_csv*).
- Use the *tools/drop-terminated-postcodes.js* script to drop from the above dataset all terminated postcodes and create a new file with the remaining ones, e.g.

	```
	node tools/drop-terminated-postcodes.js --in data/ONSPD_NOV_2014_csv/Data/ONSPD_NOV_2014_UK.csv --out data/ONSPD_NOV_2014_csv/Data/ONSPD_NOV_2014_UK_not_terminated.csv 
	```

- Download the JSON, one-file-per-postcode-sector edition of Open Addresses UK's addresses-only dataset from [http://alpha.openaddressesuk.org/data](http://alpha.openaddressesuk.org/data) and uncompress it in *data* (e.g. *data/open_addresses_database_2014-12-10-openaddressesuk-addresses-only-split.json*).
- Install all Node.js dependencies:

	```
	npm install 
	```

##Run
###If you don't have a favourite course
The example below creates a JSON file called *investigationOptions.json* that includes an list of address investigation options that are about a certain specified total run distance (including to and back, in miles) from a starting point provided as an input. 

Don't worry if you don't know your starting point, [read here](/docs/where-am-i.md). 

Note how the reduced version of the ONSPD dataset produced following the setup instructions above is given as an input.

```
> node oarunner.js --lat=51.759733 --lon=-0.577663 --distance=3 --oa=data/open_addresses_database_2014-12-10-openaddressesuk-addresses-only-split.json/ --onspd=data/ONSPD_NOV_2014_csv/Data/ONSPD_NOV_2014_UK_not_terminated.csv > investigationOptions.json
```

###If you have a favourite course
If you own a fitness device that can save your activity data to *.fit* format, the example below instead creates a JSON file called *investigationOptions-fit.json* that includes an list of address investigation options that are most suitable to the course specified in the *.fit* file provided as an input. 

```
> node oarunner.js --fit=data/fit-samples/2014-12-24-11-11-15-Navigate.fit --fitsdk=etc/FitSDKRelease13.10/ --oa=data/open_addresses_database_2014-12-10-openaddressesuk-addresses-only-split.json/ --onspd=data/ONSPD_NOV_2014_csv/Data/ONSPD_NOV_2014_UK_not_terminated.csv > investigationOptions-fit.json 
```

###The resuts
The results file is a list of postcodes suitable for surveying. Each postcode is described by:

- *postcode*: describing the postcode to be investigated, its lat/lon and the course's closest point, e.g.

	```
	{
		"pcd" : "HP4 2EB",
		"lat" : 51.760489,
		"lon" : -0.557384,
		"closestPoint" : {
			"distance" : 1.6,
			"position" : {
				"_lat" : 51.760688,
				"_lon" : -0.557431,
				"_radius" : 6371
			}
		}
	}
	```

- *relevantOaAddresses*: an array of all Open Addresses addresses that belong to that postcode, in their native OA JSON format.

- *inferredAddresses*: an array of all addresses inferred for that postcode, in the OA JSON format but for the elements that could not be determined (e.g. obviously the URI associated by OA to addresses).

The investigation options are ordered by proximity to the middle of your course. The underlying idea is that the runner's mission is to run to and back from the address to be investigated :-) 

The proposed postcodes are those whose centroid is within 50 yards (can be changed using the *--distance* command line parameter) from the running course in the specified *.fit* file. In theory, this means that investigating the addresses won't take the runner too far from her planned course. By default, not all points in the course are used, but one every ~220 yards (the *--sample* parameter). 

##Compatibility
- If you want to use *oa-runner* by specifying a favourite running course and you own a fitness device / watch, at the moment we support *.fit* files only. Testing was done using files created by a [Garmin Fēnix 2 multisport watch](https://buy.garmin.com/en-GB/GB/watches-wearable-technology/wearables/fenix-2/prod159116.html). We presume that their format and conventions (e.g. the column names in the *.csv* files that can be extracted from the *.fit*) will be consistent at least across other Garmin models, but did not test that yet. Do you own any other fitness device that uses the *.fit* format? Please offer your help in the [issues section](https://github.com/Digital-Contraptions-Imaginarium/oa-runner/issues).

##TODO
- The investigation options should not simply list the addresses that are already known to OA, but list what we're asking the runner to check! E.g. the script could do basic inference on missing PAOs and ask the runner to check if they exist for real, or confirm that they don't.
- All investigation options should be formatted in a format suitable for printing, so that the runner can take them with her. QR codes could be associated to each possible scenario being investigated, e.g. there could be one QR to say that 22 Bridge Street exists and another QR to say that it does not.

##Licence
Northing/Easting to Latitude/Longitude conversion in JavaScript code is done using Chris Veness' libraries, available at [http://www.movable-type.co.uk/scripts/latlong-gridref.html](http://www.movable-type.co.uk/scripts/latlong-gridref.html) and included in this repository for convenience. The libraries are licensed under CC-BY 3.0 and implement the algorithms described by Ordnance Survey in the "A guide to coordinate systems in Great Britain" document at [www.ordnancesurvey.co.uk/docs/support/guide-coordinate-systems-great-britain.pdf](www.ordnancesurvey.co.uk/docs/support/guide-coordinate-systems-great-britain.pdf).

QR codes are generated using Shim Sangmin's *qrcode.js* library, available at [https://github.com/davidshimjs/qrcodejs](https://github.com/davidshimjs/qrcodejs) and included in this repository for convenience. It is copyright (c) Shim Sangmin (davidshimjs) and licensed under the terms of the MIT licence.

The project uses many other Open Source libraries that are referenced in the [*package.json*]([package.json]) file but not distributed within this repository.

All other code is copyright (c) 2014 Digital Contraptions Imaginarium Ltd. and licensed under the terms of the [MIT licence](LICENCE.md).
