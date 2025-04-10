# Google Earth Engine Cloud-Masked Composites

## **Introduction** 

These scripts perform basic cloud and custom cloud/shadow masking for images taken by Landsat and Sentinel-2 satellites. It is usable in the Google Earth Engine Code Editor, using available surface reflectance imagery from the Earth Engine Data Catalog. Each script works with a set AOI to filter imagery collection and generate annual cloud-masked composites based on a selected time range (pre-set to 4-months of the Austral summer). Each final annual cloud-masked image is comprised of multiple mosaicked cloud masked images, in order of mask performance. FInal analysis-ready images are set to be exported to Google Drive.

Scripts are split by the satellite imagery that is processed. Note that Landsat scripts are grouped for compatable imagery.
 - Landsat 1, 2, and 3
 - Landsat 4, 5, and 7
 - Landsat 8 and 9
 - Sentinel-2

![unnamed-2](https://github.com/user-attachments/assets/649afda4-e1e3-4e33-8f01-412fcc052173)



## **Code logistics**

### <ins> Setup </ins>	
  -	Import AOI shapefile from assets	
  -	Apply optical and thermal scaling factors for surface reflectance imagery
         -	Scaling factors can be found on the Earth Engine Data Catalog  	  
  -	Set visualisation parameters for the final (and test) composite imagery

### <ins> Masking </ins>	
### Landsat
- Basic QA Pixel Cloud Masking with optional buffer
    - Bits are used in the L457 and L89 scripts to remove pixels idenfied as clouds, cirrus, or shaddow
- Custom Cloud Score Mask with optional buffer
    -	Identify cloudy pixels for masking based on brightness in the blue band, all visible bands, and infrared bands
    -	Detect dark pixels and identify possible shadow areas based on solar azimuth angle 
- *Note: At this time, the L123 Script only has basic QA Pixel Cloud Masking
### Sentinel 2
  - Basic QA60 Cloud Mask with optional buffer
       -	Bit 10: Clouds
       -	Bit 11: Cirrus
  -	Custom Cloud Score Mask with optional buffer
       -	Identify cloudy pixels for masking based on brightness in the blue band, all visible bands, and infrared bands
       -	Detect dark pixels and identify possible shadow areas based on solar azimuth angle 
  -	Custom Cloud Probability Mask
       - Create a mask where cloud probability (from S2 metadata) is less than 15%
       -	*Note: Since the additional CLOUDY_PIXEL_PERCENTAGE filter is not usable metadata in GEE's S2 Cloud Probability band, the two collections (S2_Harmonized and S2_CloudProb) must be manually aligned. This 'matching' is done by filtering through the times that the S2_Harmonized images are taken to match them each with the time that the corresponding derived 'Cloud Probability' image was produced. There is a small difference in time between when these two events (when the S2_Harmonized and the S2_CloudProb images are produced) occur. Hence, the getCloudProbImage function below adds a buffer of an hour before and after the harmonized images were produced to match to then filter the could probability images.

### <ins> Image collection & compositing </ins>	
  -	Collect all imagery for the AOI from specified datasets for a given time range (pre-set to 4-months of the Austral summer per year)
  -	Build cloud-filtered median composites based on specified cloud masking combinations
      - Various levels of masking are used in the mosaicking step to create a final image with as much usable surface reflectance data as possible
  -	Optional - add layers to the map for each year for visualization (used to check cloud masking performance)

### <ins> Image mosaicking </ins>	
  -	Create a mosaic by stacking the composites in an image collection
  -	This allows for representing the best-performing cloud masked images first, while filling no_data regions in the AOI with lower-performing cloud masked images
       -	This is done as better performing cloud masks can also mask useable areas with bright surface reflectance such as roads, beaches, and infrastructure

### <ins> Image export </ins>	
  -	Exports the final clipped mosaic for each year to Google Drive as a GeoTIFF
