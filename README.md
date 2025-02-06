# OpenFarm integration for Home Assistant

This integration allows fetching plants information from OpenFarm.
It creates a few service calls in Home Assistant to interact with [OpenFarm API](https://github.com/openfarmcc/OpenFarm/wiki) which
are:

* Search plant
* Get plant details

This is used as a base for the sister-integration https://github.com/Olen/homeassistant-plant which utilizes this API to
add threshold values for such as moisture, temperature, conductivity etc. based on the plant species.

## Installation

[![hacs_badge](https://img.shields.io/badge/HACS-Custom-41BDF5.svg?style=for-the-badge)](https://github.com/hacs/integration)

This can be installed manually or through HACS

### Via HACS

* Add this repo as a "Custom repository" with type "Integration"
    * Click HACS in your Home Assistant
    * Click Integrations
    * Click the 3 dots in the top right corner and select "Custom Repositories"
    * Add the URL to this GitHub repository and category "Integration"
* Click "Install" in the new "OpenFarm" card in HACS.
* Wait for install to complete
* Restart Home Assistant

### Manual Installation

* Copy the whole`custom_components/openfarm` directory to your server's `<config>/custom_components` directory
* Restart Home Assistant

## Set up

The integration is set up using the GUI. You must have a valid `client_id` and `secret` from OpenFarm to set up the
integration.
After creating an account at the OpenFarm, you can find your `client_id` and `secret`
here: https://open.plantbook.io/apikey/show/

Go to "Settings" -> "Integrations" in Home Assistant. Click "Add integration" and find "OpenFarm" in the list.
The integration validates the credentials and throws an error if they are incorrect.

## Configuration

The integration provide the following configuration options:

![image](./images/config-options.png)

### Upload plant-sensors' data

>**NOTE:** All the data is shared anonymously.

This option will enable the integration to look for plants created with
sister-integration https://github.com/Olen/homeassistant-plant

### Automatically download images from OpenFarm.

The default path to save the images is `/config/www/images/plants`, but it can be set to any directory you wish.

You need to specify an _existing path_ that the user you are running home assistant as has write access to. If you
specify a relative path (e.g. a path that does not start with a "/", it means a path below your "config" directory. So "
www/images/plants" will mean "&lt;home-assistant-install-directory&gt;/config/www/images/plants".

If the path contains **"www/"** the image_url in plant attributes will also be replaced by a reference to
/local/<path to image>. So if the download path is set to the default "/config/www/images/plants/", the "image_url" of
the species will be replaced with "/local/images/plants/my plant species.jpg".

If the path does _not_ contain **"www/"** the full link to the image in OpenFarm is kept as it is, but the image is
still downloaded to the path you specify.

Existing files will never be overwritten, and the path needs to exist before the integration is configured.

## Examples

### openfarm.search

`openfarm.search` searches the API for plants matching a string. The search result is added to the
entity `openfarm.search_result` with the number of returned results as the `state` and a list of results in the
state attributes.

```yaml
service: openfarm.search
service_data:
  alias: Capsicum
```

The result can then be read back from the `openfarm.search_result` once the search completes:

```jinja2
Number of plants found: {{ states('openfarm.search_result') }}
{%- for pid in states.openfarm.search_result.attributes %}
  {%- set name = state_attr('openfarm.search_result', pid) %}
  * {{pid}} -> {{name}}
{%- endfor %}
```

Which would produce

Number of plants found: 40

* capsicum annuum -> Capsicum annuum
* capsicum baccatum -> Capsicum baccatum
* capsicum bomba yellow red -> Capsicum Bomba yellow red
* capsicum chinense -> Capsicum chinense
  (...)

### openfarm.get

`openfarm.get` gets detailed data for a single plant. The result is added to the
entity `openfarm.<species name>` with parameters for different max/min values set as attributes.

>**NOTE:** You need to search for the exact string returned as "pid" in `openfarm.search_result` to get the right plant.

```yaml
service: openfarm.get
service_data:
  species: capsicum annuum
```

And the results can be found in `openfarm.capsicum_annuum`:

```jinja2
Details for plant {{ states('openfarm.capsicum_annuum') }}
* Max moisture: {{ state_attr('openfarm.capsicum_annuum', 'max_soil_moist') }}
* Min moisture: {{ state_attr('openfarm.capsicum_annuum', 'min_soil_moist') }}
* Max temperature: {{ state_attr('openfarm.capsicum_annuum', 'max_temp') }}
* Image: {{ state_attr('openfarm.capsicum_annuum', 'image_url') }}
```

Which gives

Details for plant Capsicum annuum

* Max moisture: 65
* Min moisture: 20
* Max temperature: 35
* Min temperature: 15
* (...)
* Image: https://.../capsicum%20annuum.jpg

### Quick UI example

Just to show how the service calls can be utilized to search the OpenPlantbook API

![Example](images/openfarm.gif)

**PS!**

This UI is _not_ part of the integration. It is just an example of how to use the service calls.

<a href="https://www.buymeacoffee.com/olatho" target="_blank">
<img src="https://user-images.githubusercontent.com/203184/184674974-db7b9e53-8c5a-40a0-bf71-c01311b36b0a.png" style="height: 50px !important;"> 
</a>
