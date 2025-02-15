# Disaster Preparedness Web Interactive

The project will explore traditional and qualitative scoring assessments of “risk/resiliency factors” associated with regional crisis preparedness and demonstrate how actionable steps in community engagement can create a different portrait of resiliency. It is based on [a pioneering project from Oregon](https://github.com/Oregon-Public-Broadcasting/earthquake-preparedness) but has been generalized to make it easy to clone and tailor to other regions.

# Dependencies

- Django Web Framework (installed automatically as part of setup)
- GeoDjango (installed automatically as part of setup)
- PostgresSQL (most package managers will auto-install this as a dependency of either of the following items)
- PostGIS
- Postgresql-server-dev-all
- GDAL (version 2.0.0 or newer)
- libjpeg-dev
- Python modules listed in [requirements.txt](./requirements.txt)
  - On a Linux machine you may need to install `python-dev` (through the Linux package manager) as a prerequisite, and if you have trouble getting `psycopg2` to install you may have better luck using the package manager's version of that module.
  - GeoDjango has other dependencies, but if you install it from a package manager they will usually be included automatically. [See this more complete list](https://docs.djangoproject.com/en/1.7/ref/contrib/gis/install/geolibs/) of required and optional additions.

# Note about Python Command Usage

Commands indicated are always just `python` but on some systems you might need to use `python3` in order to use a specific python version. If so, other commands such as `pip` have a `pip3` equivalent.

Use whichever base command is appropriate for your environment.

# Configure Dev Environment

Set up a virtual environment so that you can freely install python modules without worrying about overwriting globally installed versions. It's easy!

1. Move to the project directory (e.g. `/Applications/MAMP/htdocs/disaster-preparedness`).
2. `python3 -m venv venv` to create a new virtual environment.
3. Wait for things to happen.
4. `source venv/bin/activate` (type `deactivate` to leave). Remember to reactivate the virtual environment every time you open a terminal window and start running Python commands. Note that on some machines, you'll need to use `. venv/bin/activate` instead.
5. `pip install -r requirements.txt` or `pip3 install -r requirements.txt` to automatically install the Python dependencies listed in [requirements.txt](./requirements.txt). You may see "Failed building wheel for ..." errors for some of the modules. If so, try repeating the command. If the second run shows "Requirement already satisfied" for every module then you can safely ignore the previous error.

# "disasterinfosite" App

While management and data loading files are in this project's root directory, everything else is in `/disasterinfosite`.

## Written using:

- Postgres version: 9.4
- Python version: 3.5

## File structure

- `/disasterinfosite/data` contains the data sources (shapefiles and/or rasters) and text content (snuggets - see [Adding New Data](#adding-new-data) below for explanation) that will be loaded.
- `/disasterinfosite/migrations` contains Django-generated files that specify how to set the database up. We don't recommend editing these manually.
- `/disasterinfosite/static/css` contains all the stylesheets for this site.
- `/disasterinfosite/static/img` contains all the static images - if you want to change icons, etc, look here.
- `/disasterinfosite/static/js` contains JavaScript libraries that need to be included for various site functions.
- `/disasterinfosite/templates` and `/disasterinfosite/templatetags` contain HTML templates for the site's various pages and subsections, and Python code that processes them. Many of the simpler customizations to this site will involve editing the HTML templates.

## Installing App

This assumes `python` is the command configured to run the correct python version. Depending on your setup you may need to specify `python3`.

### Set up the "secret key" used by Django to secure forms.

- Set up an environment variable `DJANGO_SECRET_KEY` to whatever you want to set it to.
  - See http://techblog.leosoto.com/django-secretkey-generation/ for an example approach.
  - On Mac/Linux: `export DJANGO_SECRET_KEY="gibberishrandomstring"`

### Set up the database

1. Set up Postgres with PostGIS:

- To install PostGIS on a Mac using Homebrew: `brew install postgis`. Here are [PostGIS install instructions for Ubuntu](https://trac.osgeo.org/postgis/wiki/UsersWikiPostGIS21UbuntuPGSQL93Apt).
- The Mac or Ubuntu instructions will also install Postgres if you don't already have that.
- Run `brew info postgres` to see options for starting Postgres - you can have it start automatically when your computer starts, or not.
- Homebrew sets Postgres up with one user to start with, and that user is you. You should probably make a separate user for Django. If you want your user to be named `django`, do `createuser django --password`. You will then get a prompt for the password. Use only letters and numbers in the password, because you'll need to use it in a URL later.
- Linux installers don't necessarily create any users! You may need [these instructions](https://help.ubuntu.com/community/PostgreSQL#Basic_Server_Setup) and to run the next few Postgres commands as `sudo -u postgres`

2. Clone repo.
3. Create a Postgres database on the Postgres server, and install PostGIS to it.

   ```shell
   createdb [DBNAME]
   psql -d [DBNAME] -c "CREATE EXTENSION postgis; CREATE EXTENSION postgis_raster;"
   ```
    _[detailed instructions for reference](http://postgis.net/docs/manual-2.1/postgis_installation.html#create_new_db_extensions)_

4. In order to run unit tests, your user will need to be able to create and delete databases, since the test framework creates (and destroys) a new test DB for each test run. You can accomplish this using `psql -d [DBNAME] -c "ALTER USER [USERNAME] SUPERUSER;`"
5. Set up an environment variable `DATABASE_URL` that will be used by the Django Database URL app to load our database.

- example on Mac/Linux: `export DATABASE_URL="postgres://USER:PASSWORD@HOST:PORT/DBNAME"` where the USER & PASSWORD are the django account you created above in postgres, and the default HOST:PORT is localhost:5432 .

6. `source venv/bin/activate` or `. venv/bin/activate` if you haven't already activated the virtualenv this session.
7. Run `python manage.py migrate` to initialize the database's structure.

### Add API Keys

`disasterinfosite/templates/index.html` loads the Mapquest API with its own API key. This is fine for initial testing, but you should get your own key before deployment. Go to https://developer.mapquest.com/ to do so, and then edit [these two lines](https://github.com/hazard-ready/disaster-preparedness/blob/master/disasterinfosite/static/js/app.js#L20-L21).

### Load some data

0. `source venv/bin/activate` or `. venv/bin/activate` if you haven't already activated the virtualenv this session.
1. Unzip `data.zip` inside disasterinfosite, so that the data is in `disasterinfosite/data`. This data includes some sample shapefiles and related data for Missoula County, Montana, USA, to get you started. See below for instructions on replacing this with your own data.
1. `python import.py` to process the data and update some Django code to fit. For each data source, the script will prompt you for two things:
   - Which field to use to look up snuggets (see [Adding New Data](#adding-new-data) below for definition). If there is a field named `lookup_val`, that will be used by default. If you use the example `data.zip` provided in this project, use the field name `FEMADES` for `Flood_FEMA_DFRIM_2015`.
   - Whether you want to group the content from this data source in a tab with content from any others. If you want a dataset to have its own unique tab, just press return at this prompt. If you want to group 2 or more datasets together under 1 tab (e.g. if you have a shapefile for wildfire probability and another with historical wildfire information), just enter the same group name for both at this prompt. Note that these group names are only used in the code--headings displayed to the user come from the "snugget" file loaded in step 6 below--and should contain only letters, no spaces or punctuation.
1. `python manage.py makemigrations` - this and the next 2 steps combined load the new data into the database.
1. `python manage.py migrate`
1. `python manage.py shell`
   1. [inside the shell that this opens] `from disasterinfosite import load`
   2. `load.run()`
   3. `import snugget_load`
   4. `snugget_load.run()`
   5. `import prepare_load`
   6. `prepare_load.run()`
   7. `exit()` [to go back to the normal command line]
      The parts that have `snugget_load` and `prepare_load` are to import text that will be displayed in the site. See [Adding New Data](#adding-new-data) below for an explanation of "snuggets" and the format of this file.
1. If this is your first time through, or you emptied the database before loading new data: `python manage.py createsuperuser` and follow the instructions to add a Django admin user
1. If you don't already have web hosting set up, you can test your work on localhost:8000 with `python manage.py runserver`

# This instance of Hazard Ready uses Webpack to bundle its static files. For that reason, you need these additional steps to set it up:

1. Make sure that you have [Node and NPM installed](https://www.npmjs.com/get-npm)
1. In the same directory that contains `package.json`, run `npm install`
1. Run `npm run webpack`

#### Environmental Variable Permanence

On Linux/Mac, as soon as you close your shell you lose those nice complicated database urls.
Save them to your `.bash_profile` or equivalent.

### Create a user and visit the admin screen to verify

1. `python manage.py createsuperuser`
2. `python manage.py runserver` to run a development server on localhost:8000 or see below for how to deploy via Apache.
3. Visit http://server.ip/admin and log in with new user.
4. You should see two lists: `Authentication and Authorization` and `Disasterinfosite`. The first one contains information about the Django superuser, as well as users who sign up for this site.
5. If you have a problem loading the site while logged in as a superuser, it may be because the app is looking for additional information that it usually stores when it creates a user - but Hazard Ready didn't create that user, Django did. To fix that, go to http://server.ip/admin/auth/user/ and select that user, then click 'Save'. You don't have to change anything.
5. `Disasterinfosite` has content that you can and should edit! They are bits of text and other information that show up on this site, as well as information about how to display certain things on the site. See the [Django Admin Settings](#django-admin-settings-and-what-they-mean) section for more details.
### Deploying to the web via Apache

1. Install a version of `mod_wsgi` that is compiled for Python 3. On Debian/Ubuntu you can do this with `aptitude install libapache2-mod-wsgi-py3`. On other systems it may be easier to use `pip` as per [these instructions](https://pypi.python.org/pypi/mod_wsgi).
2. Use [these instructions](https://docs.djangoproject.com/en/1.9/howto/deployment/wsgi/modwsgi/) to configure Apache. Note in particular:
   1. You'll need `WSGIScriptAlias` to point to `disasterinfosite/wsgi.py`
   2. You'll need to apply the "Using a virtualenv" addition.
   3. You'll need to set up a `/static/` alias pointing to `disasterinfosite/static`
   4. Depending on your server configuration, you _may_ also need to set up a redirect rule to add trailing slashes to URLs, to get the static files (CSS, images etc) included.
   5. You may also need to alter the `STATIC_URL` constant in `settings.py` based on your server setup.
3. Set up the environment values from above (`DJANGO_SECRET_KEY` and `DATABASE_URL`) for all users by putting their declarations in `/etc/apache2/envvars` and restarting Apache.
4. Don't forget to run python manage.py collectstatic to get your static files where we expect them to be!
5. There should be directories called 'photos' and 'data' in disasterinfosite/staticfiles/img. This is where images go when you upload them via Django Admin, under 'Photos of Past Events' and 'Data Overview Images'. In order for that upload to work, you need to create them if they aren't present, and change the owner (chown) those directories to whatever user Apache is running as (www-data, perhaps).

### Use foreman to run the server Heroku-style

_Not tested by the current maintainers_

- `foreman start`
  - Any errors that pop up are probably from missing modules or missing environmental variables. Read the errors!

## Adding new data

### What you need

1. At least one shapefile or raster, meeting the requirements listed below.
2. Some text content to display when a user chooses a location in one or more of your shapefiles. In this project, the text content is referred to as **snuggets**, from "story nuggets".

#### Requirements for a shapefile

1. Each shapefile's attribute table must contain a column with a unique identifier for each set of text to display (e.g. all the areas for which you want to display "Expected ground shaking: severe" have one ID, and all the areas for which you want to display "Expected ground shaking: moderate" have another). This column will be used to look up text when a user selects a location.
2. That column's name must comply with the [Django field name restrictions](https://docs.djangoproject.com/en/1.9/topics/db/models/#field-name-restrictions), including not being one of the [Python 3 reserved words](https://stackoverflow.com/questions/22864221/is-the-list-of-python-reserved-words-and-builtins-available-in-a-library/22864250#22864250). For example, if the column is called `id`, `object`, `map`, `property` or `type`, you'll have to rename it.
3. It doesn't matter which coordinate reference system the shapefile has been saved with, but if you're making them yourself then we recommend using EPSG:4326, because the import pipeline will reproject it to that anyway.
4. If you have multiple shapefiles, clip them all to cover the same area. Otherwise, if users click on a location that is covered by some shapefiles but not others they will see partial data without a clear explanation that there is missing data.
5. Multiple shapes may overlap, but each shape may only have one value for the lookup field. If you have multiple shapes with the same lookup value, the import process will combine them.

#### Requirements for a raster file

1. The format must be GeoTIFF, with a `.tif` file extension.
2. Band 0 must contain a unique identifier for each set of text to display. If the file contains multiple bands, all but the first will be ignored.
3. Band 0 must contain unsigned integers no larger than 254 (they'll be stored as a single byte, real numbers will be rounded off to the nearest integer, and 255 is reserved for NODATA).
4. The values in Band 0 must be higher for more serious warnings. This is because the file will be reprojected to EPSG:4326 during import, and values are rounded up in the reprojection. As long as higher values are the more serious warnings, this will not create a risk of people seeing an inappropriate "all clear" message for their location.
5. Each data source must be one single `.tif` file. Large files will be tiled automatically during the load but the data import pipeline does not currently have the ability to combine rasters.

### Fully automated pipeline

If the structure of your text content is simple enough, you can import shapefiles/rasters, snuggets and preparedness actions automatically without having to do much manual work. We recommend using this pathway if possible, because it makes moving the site to a new server significantly easier.

#### Importing snuggets

To do this, you will need a `snuggets.xlsx` file with the same columns as the example one we've included in `data.zip`. The columns can be in any order, but the headings must be exactly as typed here:

- `heading` : A human-readable heading that describes the content of this shapefile or raster, to be displayed on the page. This will correspond to the shapefile group.
- `section` : A section name that will be displayed on the page (must not be empty)
- `shapefile` : The file name for the shapefile or raster file this row corresponds to, without the extension. For example: `EQ_GroundShaking_MostLike` for text that relates to the content of `EQ_GroundShaking_MostLike.shp`. (must not be empty; must correspond exactly to the available files)

-`txt_location` : The order in which this text will appear within its section.

-`pop_out_image` : An auxiliary sidebar image to display with this snugget, as an aside.

-`pop_out_link` : A link to go with this snugget, as an auxiliary aside.

-`pop_alt_txt` : The alt text for `pop_out_image`

-`pop_out_txt` : Text to go in the auxiliary sidebar for this snugget.

-`lookup_value` : The value of the unique identifier in the shapefile (e.g. an intensity value or a hazard classification), or the pixel value of a raster that corresponds to this. This field can be empty; if it is then the rest of this row will be applied to every available value. If this field contains a value that is not present in the data, the load script will find that, warn you, and skip loading it.

-`intensity` : Relative severity scaled from 0-100, to display graphically on the page. If this is empty, or if a value is provided for `image`, it will simply not be displayed.

-`intensity_txt` : Explanatory text for the intensity value.

-`text` : The explanatory text to be displayed in the relevant section and subsection when the user chooses a location that matches this row's criteria. If you put a url in the snugget text, like `http://www.github.com`, we'll automatically make it into a link for you.

-`image_slideshow_folder` : If this is present, the snugget loader will look for a folder inside an `images` folder at the same level as `snuggets.xlsx` (example: if `snuggets.xlsx` is in `data` and you set this value to `earthquake_photos`, the loader will look for images in `data\images\earthquake_photos`). The loader expects to find two things in that folder: images, and a file called `slideshow.csv`.

`slideshow.csv` should have two columns: `image` and `caption`. `image` is the filename of the image, and `caption` is whatever you would like the caption for that image to be.

-`video` is the url of a YouTube video that you would like to embed in this snugget.

You can have any number of sections, but every row must be a unique combination of `shapefile`, `section`, and `lookup_value`. If you define more than one row for the same permutation, only the last one in the file will actually be used. Note that this allows you to create a default value for a given section, subsection and shapefile, by having a row with `lookup_value` blank (so it applies to all values present in the data source), followed by rows with specified `lookup_value`s which will overwrite the default for that value only.

Blank rows or additional columns won't cause problems. Any row that is missing any of the required fields will be skipped and a warning will be printed.

Once `snuggets.xlsx` is ready, simply put it and the relevant data files in `disasterinfosite/data` (and remove any other files or subdirectories from there), and follow the instructions in [Load Some Data](#load-some-data) above.

#### Importing Preparedness Actions

This is the content that shows up on the Prepare page. The concept is similar to loading the snuggets - you need a `prepare.xlsx` file with the same columns as the example one we've included in `data.zip`. The columns can be in any order, but the headings must be exactly as typed here:

- `section` the title of the section, like 'Learn your hazards'
- `cost` An value representing how much it costs to take the action. Your options are:
  - 0: Free!
  - 1: $1 - $30
  - 2: $31 - $100
  - 3: $101 - $300
  - 4: more than \$301
- `image` The filename of an image to associate with the text of this preparedness action. The prepare loader will look for these files inside `images\prepare` at the same level as this file. So if this file is in `data`, the loader will look for your images in `data\images\prepare`.
- `happy` A sentence or two about how this action can reduce physical or emotional harm after an event.
- `useful` A sentence or two about how this action can be useful for other situations
- `property` A sentence or two about how this action can help protect your property
- `text` Explanatory content about this action. HTML is allowed in this field.
- `external_link` A link to visit for more information
- `external_text` The text to display for the link
- `external_icon` An icon or image to display for the link


#### Updating existing data

If you make changes to `snuggets.xlsx` you should only need to re-run `python snugget_load.py` and restart your web server.

If you make changes to the shapefiles/rasters, or change which field from the shapefiles you want to use as the ID, then before running `python import.py` you will also need to remove the `disasterinfosite/data/reprojected` and `word/data/simplified` directories that the importer had created. It uses these to avoid having to repeat the time-consuming reprojection and simplification of the shapefiles or reprojection of the rasters every time it is run, but that means changes to the files themselves won't be picked up unless they are removed.

If you have existing data that needs to be removed—perhaps because you are replacing our sample data, or retiring a dataset you previously used—you may have to clear the database first. To do this:

1. `psql -d [DBNAME] -c "DROP SCHEMA public CASCADE; CREATE SCHEMA public; CREATE EXTENSION postgis;"`
2. `python manage.py migrate` - if this step throws errors, delete all the .py files in `disasterinfosite/migrations` **except** `__init__.py` and `0001_initial.py`, and try again.

Then continue with the instructions in [Load Some Data](#load-some-data) above.

### Working with more complex text templates

You may want to use multiple fields from a shapefile to fill in blanks from a template, such as "In year [YEAR] the [FIRENAME] fire burned [AREA] acres here". The automated import pipeline is not sophisticated enough to do this for you, so you have two options:

#### If you can edit the shapefile

Using QGIS or ArcGIS, add two columns to the shapefile: one with a lookup value composed of all the variables you're using (e.g. `[FIRENAME]_[YEAR]_[AREA]`), and one with the complete text. You can create both of these using calculdated fields in either program. Then copy-paste the attribute table into Excel or an equivalent, and use the complete texts to populate the `text` column of `snuggets.xlsx` and the lookup values for the `lookup_value` column. With this, you can use the automated pipeline to do the rest.

#### If you can't edit the shapefile, or are more comfortable editing code

Take a look at `disasterinfosite/models.py`, `disasterinfosite/load.py` and `disasterinfosite/admin.py` after running the automated pipeline on some sample data, and write appropriate equivalents for all of the generated code (marked by prominent comments) that fit your data and text model. You may also need to edit `disasterinfosite/templates/found_content.html` which is the page template to be displayed when there is at least one snugget available for a location. Then run just the `manage.py` parts of the [Load Some Data](#load-some-data), and use the Django admin panel to enter snuggets by hand.

If you have some data that fits that automated import model and some that does not, you can combine the two. Just watch for three things:

1. You'll have to reproject the shapefiles that aren't going through the import pipeline to EPSG:4326 yourself.
2. Put the shapefiles that aren't being manually imported somewhere other than `disasterinfosite/data` to keep them out of the automated pipeline.
3. Be very careful to avoid putting any of your manually edited code between the `# GENERATED CODE GOES HERE` and `# END OF GENERATED CODE BLOCK` comment pairs in the Python files, because that part gets overwritten by `import.py` each time.

## Django Admin settings and what they mean

###### Past Events Photos
Upload photos to show in a photo gallery in the search results, under Past Events. Make sure that the heading you enter here matches the heading that the photos will appear under.

###### Data Overview Images
In the box at the bottom of every page, there's a section called 'Quick Data Overview'. That's where these will show up, as links that open in a new tab or window. The link_text field is what the link says, like 'Earthquakes: Distance from a Fault', and you can upload the appropriate image here.

###### Shapefile Groups
When you imported data, you were asked for a group name for your shapefiles, so that you can present, say, all your earthquake data togther, all your volcano data together, and so on. This is the place where you can choose display names for those groups that show up on the site, and configure the order in which they will appear on the page with all the content. You can also add a note at the top of the section that they appear in.

###### Site Settings
Basic information about this site and who created it. This stuff shows up in the page headers and footers, as well as in the introductory text on the landing page. The `about text` and 'who made this' sections especially deserve lots of details, and the Data Download link is if you'd like to share the data that you used to create this site. The site title is the big text at the top.

###### Snugget Sections
Inside shapefile groups, [snuggets](#importing-snuggets) are also in groups by section name. Here, you can choose a display name for those sections, an order in which it will appear inside the shapefile group, and whether it is always shown, or collapsed into a header that just shows the display name.

# Supporting multiple languages

## The app
To support a multi-language site, you need to provide message files for the Django views, by the usual means of `makemessages` and `compilemessages`.
In addition, there are many bits of text that are configurable through Django Admin. To translate those, uncomment the relevant lines in settings.py, admin.py and translation.py. Then run `makemigrations` and `migrate`. You should then have fields for multiple languages exposed in Django Admin.
For more information, see the [Django-ModelTranslation docs](http://django-modeltranslation.readthedocs.io/en/latest/index.html)

## Snuggets and Preparedness Actions
When you uncommented the relevant lines of code in the previous section, you enabled translating snuggets and preparedness actions. That means that if you put some specially named columns in your snugget and prepare spreadsheets, Hazard Ready will read in those translations as well during the load process. To translate a column, make another column with the same name, `-`, and then the relevant language code. So the Spanish translations for the `text` column in the `snuggets.xlsx` will be `text-es`, and it will be automatically associated with the correct snugget. You can have as many languages as you want for each column.

The following columns will be automatically translated in this way:
- `snuggets.xlsx`
  - `text`
  - `pop_out_txt`

- `prepare.xlsx`
  - `section`
  - `text`
  - `happy`
  - `useful`
  - `property`
  - `external_text`

