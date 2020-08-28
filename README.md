# ePaper Parking Display
A display that shows your car's parked location

## Full Installation Instructions
### Setting up the Pi
1. [Download the latest Raspberry Pi Imager](https://www.raspberrypi.org/downloads/) and flash to an SD card.
   - Use Raspbian Buster with Desktop
2. Edit the wpa_supplicant.conf file and save it to the boot directory.
   - Edit the "ssid" and the "psk" to be your wifi name and password
   - Edit the "country" to the two-digit ISO code for your country
3. Add a blank file named "ssh" (no file extension) to the boot directory to enable SSH.
4. Eject your SD card from your computer and insert it into your Pi. Plug the Pi in to power.
5. Use an app such as [Fing](https://www.fing.com/) to scan your wifi network and identify the IP address of your new Pi.
6. Connect to the Pi using SSH.
   - [More info can be found here](https://www.raspberrypi.org/documentation/remote-access/README.md). 
   - Don't forget to change your password using the `passwd` command.
7. Expand the file system using `sudo raspi-config --expand-rootfs`.
8. Enable SPI using `sudo sed -i s/#dtparam=spi=on/dtparam=spi=on/ /boot/config.txt`.
9. Reboot Pi to apply the changes.
10. Update and upgrade the Pi using `sudo apt update` and then `sudo apt full-upgrade`.

### Copy files and install required packages
11. Create the parking directory on the Pi using `mkdir parking`.
12. Copy the github files into the parking directory.
13. Install required packages using the following commands:
    - `pip3 install pyowm`
    - `pip3 install Pillow==6.2.2`

### Set up required dependencies for project
14. Set up Google Sheets
    - [Follow the quick start instructions](https://developers.google.com/sheets/api/quickstart/python) on a desktop computer.
    - Install the required packages on your Pi using `pip3 install --upgrade google-api-python-client google-auth-httplib2 google-auth-oauthlib`
    - Use [SCP](https://www.raspberrypi.org/documentation/remote-access/ssh/scp.md) to copy the token from your desktop computer to the parking directory on your Pi. Name the credentials `token.pickle`.
    - In display_image.py, set `credentials_file_path` as the file path to your `token.pickle` file.
    - Create a Google Sheet that will house your coordinates. Name the tab 'current' and place your coordinates at the cell B1.
    - In display_image.py, set `sheet_id` to be the sheet id, which can be found in the URL of the sheet.
15. Get the driver for your display.
    - Download your display's driver from [here](https://github.com/waveshare/e-Paper/tree/master/RaspberryPi%26JetsonNano/python/lib/waveshare_epd).
    - Copy the file into your parking directory.
    - In driver file, change the second line that says `from . import epdconfig` to be `import epdconfig`.
    - In display_image.py, change `epd_7_in_5_v3_colour` in the first line to be the name of your driver file (without the .py).
    - Note the resolution listed under `# Display resolution` in the driver. We will use this later.
16. Get the map image
    - Create a map image in [Mapbox Studio](https://studio.mapbox.com/).
    - I won't walk you through how to create this, but I recommend starting with the black and white template. You can turn off various layers and resize and change the color of roads and labels.
    - Use the "Print" feature in the tool bar and fiddle with the width/height and resolution until it says that the dimensions are exactly equal to the resolution of your display. My settings were as follows:
    ```
    Dimensions: 528 px x 880 px
    Width: 8.66 in
    Height: 14.43 in
    Resolution: 61 ppi
    ```
    - After you export the png file, double check the dimensions of the picture. Make sure they are exactly the dimensions of your epaper display. If they are even one pixel off, nothing will display on your screen and there will be no error message - a very frustrating thing to troubleshoot.
17. jfsljsd the map image.
    - In order to convert from lat and lon coordinates to pixels on your map, you will need to find the pixel coordinates and the location coordinates for two points and set them in the display_image.py file.
    - Set the first pixel coordinate and lat/lon coordinates as `pix1` and `coord1`. Set the second as `pix2` and `coord2`.
18. Create the geojson file.
    - Create a geojson file with [geojson.io](http://geojson.io/) full of lines that follow every street within the bounds of your map image. If you draw the bounds of your image as reference, don't forget to delete them before saving the file. Also add a "name" property and name each street.
    - Each street must be a single straight line defined by only two points. If your streets have curves, you must make many individual straight lines.
    - Save the geojson file and copy it to your parking directory.
    - In `display_image.py` set `geojsonfile` to be the file path and name of your geojson file.
    - You can skip this part, but you'll need to edit `display_image.py` to remove all references to this part.
19. Add display_image.py to the crontab file.
    - Use `crontab -e` to edit the file.
    - Add `*/10 * * * python3 /home/pi/parking/display_image.py` to the bottom of the file.

### Create your tasker plugin to populate the location
20. Create a task that runs the following:
    - Get Location v2
    - Use the Spreadsheet Update plug-in to write %gl_coordinates to cell B2 on the tab called "current"
    - I also update a second tab in the sheet to make a log with the date/time and the coordinates so I can go back and troubleshoot if necessary.
21. Create a profile to trigger the task
    - Profile should be triggered when BT is connected. Set the BT to your car's BT.
    - Connect the get location task you just created.
    - Move the task to "Exit" so it only runs when bluetooth is disconnected.
