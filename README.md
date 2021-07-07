## About

The [NIfTI format](https://nifti.nimh.nih.gov) is popular with scientists for storing brain imaging data (MRI, PET, CT). It is much simpler than the [DICOM format](https://en.wikipedia.org/wiki/DICOM) used by medical imaging manufacturers. However, the relationship between the voxel-values stored on disk and the real-world position of that voxel in space can be confusing. This repository provides examples of the same image, permuted with all 48 lossless permutations of spatial rotations and flips. Since the [rotation is stored in the NIfTI header](https://brainder.org/2012/09/23/the-nifti-file-format/), each image should appear identical to a tool that correctly interprets the transform, even though each stores the data to disk differently.

## Orthogonal Spatial Rotations

Neuroimagers tend to refer to the three anatomical axes of the brain using [Talairach coordinates](https://en.wikipedia.org/wiki/Talairach_coordinates):

 - X `Left`/`Right` with values increasing rightward.
 - Y `Posterior`/`Anterior` with values increasing anteriorly.
 - Z `Inferior`/`Superior` (aka `Foot`/`Head`)  with values increasing superiorly.

Note that [DICOM and NIfTI have different polarities for spatial coordinates](https://www.nitrc.org/plugins/mwiki/index.php/dcm2nii:MainPage#Spatial_Coordinates). Specifically, with DICOM images X increases leftward, Y increases posteriorly and Z increases superiorly.

Note that we typically do not use the terms `Caudal`/`Rostral` and `Ventral`/`Dorsal` as these [are complicated for bipeds](https://en.wikipedia.org/wiki/Anatomical_terms_of_location) (e.g. the `dorsal` direction is different for the spine versus the brain).

When images are initially acquired, the origin X×Y×Z = 0×0×0 is arbitrary (isocenter for MRI, table-center for CT), and the directions are relative to the scanner bore. Once we reorient our scans to standard space, the origin should be near the [anterior_commissure](https://en.wikipedia.org/wiki/Anterior_commissure) and the spatial axes should be aligned with the brain's anatomical orientation.

3D NIfTI volumes have a fixed number of rows, columns and slices. For example, a fMRI scan might have a 64×64×36 Columns×Rows×Slices for a total of 147456 voxels. These will be saved on disk as a 1D array with zero-based indices 0..147455. NIfTI refers to columns, rows and slices as i, j, and k respectively. Therefore, indices 0..63 refer to the spatially adjacent columns (i) of the first row (j) of the first slice (k). Likewise, voxels 64..127 refer to adjacent columns (i) of the second row (j) of the first slice.

The three spatial dimensions X (LR), Y (PA) and Z (IS) can be in stored as 48 possible permutations. These are the six permutations possible for XYZ dimensions (shown as rows in the columns below) times the eight possible polarities (whether an axes is flipped or not, shown as the columns in the table). This is shown in the table below, where each cell shows a mapping from Column×Row×Slice (I×J×K) to X×Y×Z. For example `R×A×S` means that columns index left-to-right, rows index posterior-to-anterior and slices index from inferior-to-superior:
 
| +++   | ++-   | +-+   | +--   | -++   | -+-   | --+   | ---   |
| ----- | ----- | ----- | ------| ----- | ------| ------| ----- |
| R×A×S | R×A×I | R×P×S | R×P×I | L×A×S | L×A×I | L×P×S | L×P×I |
| R×S×A | R×S×P | R×I×A | R×I×P | L×S×A | L×S×P | L×I×A | L×I×P |
| A×R×S | A×R×I | A×L×S | A×L×I | P×R×S | P×R×I | P×L×S | P×L×I |
| A×S×R | A×S×L | A×I×R | A×I×L | P×S×R | P×S×L | P×I×R | P×I×L |
| S×R×A | S×R×P | S×L×A | S×L×P | I×R×A | I×R×P | I×L×A | I×L×P |
| S×A×R | S×A×L | S×P×R | S×P×L | I×A×R | I×A×L | I×A×R | I×P×L |

The included NIfTI images demonstrate all of these possibilities.

## Oblique Spatial Rotations

Be aware that raw images may not be acquired orthogonal to the scanner bore. Likewise, the image may be oblique with anatomical space. These fine rotations can be stored in the NIfTI header, so calling a scan alignment `R×A×S` may be only an approximation.

## S-Form and Q-Form

The NIfTI header can store [two independent spatial transforms](https://nifti.nimh.nih.gov/nifti-1/documentation/nifti1fields/nifti1fields_pages/qsform.html): the quaternion-based Q-Form and the matrix-based S-Form. For example, the Q-Form might provide the transformation to the raw scanner space, while the S-Form might provide a linear transform to standard anatomical space. Be aware that the matrix can store a [shear](https://discourse.itk.org/t/saving-non-orthogonal-volume-in-nifti-format/2760/21) while the quaternion can not (i.e. quaternions can only represent [an orthogonal matrix](https://en.wikipedia.org/wiki/Rotation_matrix)). Quaternions have clear benefits over matrices: they are more compact, it is simple to smoothly interpolate between them, and the inability to store shears prevents artificial shears due to rounding errors. On the other hand, matrices benefit by being able to store shears and being simpler for most people to visualize. Unfortunately, some programs give precedence to the Q-Form while others give precedence to the S-Form. For this reason, it is probably wise to use the same transform for both the S-Form and Q-Form (and only store orthogonal volumes in NIfTI format).

## Links

 - [Anderson Winkler's introduction to the NIfTI header](https://brainder.org/2012/09/23/the-nifti-file-format/).
 - [BrainVoyager notes on coordinate systems](https://www.brainvoyager.com/bv/doc/UsersGuide/CoordsAndTransforms/CoordinateSystems.html).
 - [FSL notes on spatial orientation](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/Orientation%20Explained).
 - [dcm2niix notes on DICOM versus NIfTI spatial coordinates](https://www.nitrc.org/plugins/mwiki/index.php/dcm2nii:MainPage#Spatial_Coordinates)





 
 
