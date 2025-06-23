# OIC-150_SATB2-GFP_circ3915-3XFLAG_Fluorescence_Quantification_Report
Total hours: 12.5

Github Repository: https://github.com/vaioic/OIC-150_SATB2-GFP_circ3915-3XFLAG_Fluorescence_Quantification

## Authorship and Methods
Research supported by the Optical Imaging Core should be acknowledged and considered for authorship. Please refer to our [SharePoint page](https://vanandelinstitute.sharepoint.com/sites/optical/SitePages/Acknowledgements-and-Authorship.aspx) for guidelines. 

Please include our RRID in the methods section for any research supported by the OIC. RRID:SCR_021968

### Sample Acknowledgement
We thank the Van Andel Institute Optical Imaging Core (RRID:SCR_021968), especially [staff name], for their assistance with [technique/technology]. This research was supported in part by the Van Andel Institute Optical Imaging Core (RRID:SCR_021968) (Grand Rapids, MI).

## Summary of Request
Segment nuclei in 3D and measure the mean fluorescence intensity of all channels.

## Brief summary of analysis pipeline
Used [CellPose](https://github.com/MouseLand/cellpose/tree/v3.1.1.2) and [scikit-image](https://scikit-image.org/) to segment the nuclei and measure the mean fluorescence intensity of all channels in the image.

## Data
Images were 3D, multi scene. multi channel czi files acquired on the Zeiss LSM880 with AiryScan. Number of scenes (3-5 scenes) and channels were variable among the dataset, with some images having 3 channels and others having 4. Analysis pipeline had to be agnostic to the number of channels and scenes in each image.

## Analysis Pipeline
A Python script was written to segment and analyze the data.

Images were read in with [`aicspylibczi`](https://github.com/AllenCellModeling/aicspylibczi). Preprocessing of the images included removing empty dimensions, and reordering the image dimensions with [`numpy`](https://numpy.org/doc/stable/index.html) to be compatible with downstream steps.

The nuclei channel was copied and blurred with a gaussian kernel prior to running through CellPose to detect nuclei. The `nuclei` model was used and run per z plane with stitching of overlapping objects to create the final nuclei labels in 3D. 

scikit-image was then used to measure the mean fluorescence intensity of each channel. The measurements were written out as a csv using [`pandas`](https://pandas.pydata.org/docs/index.html).

## Output

Example output of measurements:

|    |   Unnamed: 0 |   label |   intensity_mean-0 |   intensity_mean-1 |   intensity_mean-2 |   intensity_mean-3 |
|---:|-------------:|--------:|-------------------:|-------------------:|-------------------:|-------------------:|
|  0 |            0 |       1 |            1.97191 |            56.1652 |            113.268 |            3136.62 |
|  1 |            1 |       2 |            2.12117 |            57.1292 |            120.857 |            4402.19 |
|  2 |            2 |       3 |            2.18829 |            65.8071 |            134.491 |            4176.6  |
|  3 |            3 |       4 |            1.88749 |            52.9857 |            115.121 |            4059.73 |
|  4 |            4 |       5 |            2.09288 |            59.4496 |            125.634 |            3239.66 |

The Label column is the object (nuclei) ID. The Unnamed column can be ignored.

Mean fluorescence intensity was measured for each channel in 3D. Channels are named by their index value (0,1,2,etc.) and are in the same order as how the channels were collected.

For 3-channel images the key is:
|Value | Channel|
|------|--------|
|intensity_mean-0 | Red | 
|intensity_mean-1 | Green | 
|intensity_mean-2 | DAPI | 

For 4-channel images the key is:
|Value | Channel|
|------|--------|
|intensity_mean-0 | Far Red | 
|intensity_mean-1 | Red | 
|intensity_mean-2 | Green | 
|intensity_mean-3 | DAPI |

The blurred nuclei channel, separate scenes, and nuclei masks were all saved as tiff files. All measurements were saved as csv files and can be opened with Excel, R, Pandas, or another data frame software. Each tiff and csv are named after the parent image with the scene number appended to the end.

## Notes
The nuclei touching the XY borders of the images should be excluded from the analysis as they are not fully representative of the expression. This will require visual inspection of the label images to identify which ones to exclude. 'skimage.segmentation.clear_border` can be used to automate this step, but most nuclei also touched the first or last z-slice and would therefore be removed as well.