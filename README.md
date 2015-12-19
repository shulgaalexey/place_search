# How to Use the Tizen Native Place API in 4 Steps

*Since Tizen 2.4*

## What is the Place API?
**Place API** allows you to discover and explore point of interest (POI) information.

The place API is one of the Maps Services provided by the Tizen Native Location Framework.

<img src="https://github.com/shulgaalexey/place_search/blob/master/doc/maps_service.png" alt="Tizen Native Maps Service API" style="width:500px"/>

To start using the Place API:

1. Create an empty Tizen native application.
2. Start the Maps Service.
3. Prepare a place search filter.
4. Run a place request.

## Prerequisites
*This document assumes that you already have basic knowledge in Tizen development. For basic information, see* https://developer.tizen.org/development/getting-started/preface

The Maps Service API requires a security key issued by the maps provider.

In case of HERE maps, the security key is a concatenation of app_id and app_code, generated on 
https://developer.here.com/plans/api/consumer-mapping according to your consumer plan.

```
“your-security-key” is “app_id/app_code”
```

Note: Make sure your device or emulator has a valid internet connection.

To ensure the Maps Service API execution, set the following privileges:
 * http://tizen.org/privilege/mapservice
 * http://tizen.org/privilege/internet
 * http://tizen.org/privilege/network.get

<img src="https://github.com/shulgaalexey/place_search/blob/master/doc/set_privileges.png" alt="Set Privileges" style="width:500px"/>


## 1. Create an Empty Tizen Native Application
In the IDE, create an empty application using the basic UI application template, and run it on an emulator or a device.

<img src="https://github.com/shulgaalexey/route_search/blob/master/doc/create_empty_prj.png" alt="Create Empty Tizen Native Project" style="width:500px"/>

The "Hello Tizen" label appears on the screen at application startup.

NOTE: Get familiar with instructions on how to create an empty application at  https://developer.tizen.org/development/getting-started/native-application/creating-your-first-tizen-application.


## 2. Start the Maps Service
### 1. Include the Maps Service API header file in your application:

```C
#include <maps_service.h>
```

NOTE: This inclusion allows you to use all native Maps Service API functions and features. For more details, see https://developer.tizen.org/development/api-references/, and go to 2.4 API References -> Native Application -> Mobile Native -> Native API Reference -> Location -> Maps Service.

### 2. Add a Maps Service handle to the appdata_s structure:

```C
typedef struct appdata {
	Evas_Object *win;
	Evas_Object *conform;
	Evas_Object *label;
	maps_service_h maps; // Maps Service handle
} appdata_s;
```

### 3. Create the Maps Service instance in the app_create() function:

```C
static bool
app_create(void *data)
{
   appdata_s *ad = data;
   create_base_gui(ad);

   // Specify the maps provider name
   if (maps_service_create("HERE", &ad->maps) != MAPS_ERROR_NONE)
      return false;

   // Set the security key issued by the maps provider
   maps_service_set_provider_key(ad->maps, "your-security-key");

   return true;
}
```

### 4. When no longer needed, destroy the Maps Service instance in the app_terminate() function:

```C
static void
app_terminate(void *data)
{
   // Release all resources
   appdata_s *ad = data;
   maps_service_destroy(ad->maps);
}
```


## 3. Prepare a Place Search Filter
Prepare a search filter to query for places of the "transport" category in the app_create() function:

```C
maps_place_filter_h filter = NULL;
maps_place_category_h category = NULL;
maps_coordinates_h coords = NULL;

// Create a place search filter
maps_place_filter_create(&filter);
	
// Create a transport category
maps_place_category_create(&category);
maps_place_category_set_id(category, "transport");

// Set the place category to the search filter
maps_place_filter_set_category(filter, category);
	
// Create the central coordinates of the search area
maps_coordinates_create(41.9, 12.5, &coords);
```

Note
For information about the categories for HERE maps, see https://developer.here.com/rest-apis/documentation/places/topics/categories.html.


## 4. Run a Place Request

To run a place request:

### 1. Add the place request into the app_create() function:

```C
// Use the Place API
int request_id = 0;
maps_service_search_place(ad->maps, coords, 500, filter, NULL, search_place_cb, ad, &request_id);
```

After creating the request, release any temporary data:

```C
maps_coordinates_destroy(coords);
maps_place_category_destroy(category);
maps_place_filter_destroy(filter);
```

### 2. Implement the place callback:

```C
static bool
search_place_cb(maps_error_e error, int request_id , int index, int total,
		maps_place_h place, void *user_data)
{
   char *name = NULL;
   int distance = 0;

   maps_place_get_name(place, &name);
   maps_place_get_distance(place, &distance);

   char place_info[0x80] = {0};
   snprintf(place_info, 0x80, "Place \"%s\" is in %d meters", name, distance);

   appdata_s *ad = user_data;
   elm_object_text_set(ad->label, place_info);

   // Release the place handle
   maps_place_destroy(place);
   free(name);

   // If return true, you receive other places,
   // corresponding to the search parameters
   // In this example, 1 place is enough
   return false;
}
```

### 3. Run the application.

At first, the familiar "Hello Tizen" line appears. A moment later, however, it changes to "Place "Stazione Termini" is in 152 meters".
This indicates the transportation place is the Stazione Termini station in Rome.


## Reference
https://developer.tizen.org/development/tutorials/native-application/location/maps-service#search_place

https://developer.tizen.org/community/tip-tech/how-use-tizen-native-place-api-4-steps

