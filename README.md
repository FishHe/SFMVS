# Algorithm for SFM and MVS
> This markdown is maintained by FishHe(https://fishhe.github.io/).

This library uses a PIMPL model to isolate from 3rdparty header files. So you're released from various 3rdparties. Just include **SFMVS.h** to your project to use this library.

All the sfm and mvs functions along the pipline are packaged into asynchronous classes. One can easily use these by:

```cpp
// create the options for a specific step in the pipline
SfmvsPiplineOptions functionOps(...);
SfmvsPipline function(functionOps);

// async call
function.Start();
// wait
function.Wait();
```

For example, we use feature extractor:

```cpp
std::string project_path = "H:/RPFTest/";
std::string image_path = "H:/RPFTest/images";

Sgg4DRecon::FeatureExtractionOptions fe_ops(project_path, image_path);
Sgg4DRecon::FeatureExtractor fe(fe_ops);
fe.Start();
fe.Wait();

// the process can be get from another thread
std::thread t([&fe] {
  while (true)
  {
    std::cout << "Main Test: " << fe.GetProcess() << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
  }
});
```

In the FeatureExtractionOptions, one can define more details.


## Directory Structure

### Project Structure

This project should follow this structure.

- **project root**
  - **images** Original images.
  - **sparse** Sparse models.
    - **0** Sparse Model 0
	    - **modelfiles** (cameras, images and points3D information)
	- **1** Sparse Model 1
	- **...**
  - **dense** Dense models.
    - **0** Dense Model from Sparse Model 0
	  - **images** Undistorted images from Sparse Model.
	  - **sparse** Undistorted Sparse Model files.
	  - **stereo** Stereo intermediate results.
	- **1** Dense Model from Sparse Model 1
	- **...**
  - **colmap.db** Colmap database.

### Fusibile Structure

We use following abbreviations:

    BN: block number (int)
    IN: image number (int)



- **fusibile** fusibile point folder
  - **images** original image folder
    - **imageName1.png** original image
    - **imageName2.png** original image
    - ...
  - **cams** camera folder
    - **imagePrefix1.png.P** camera file for imagePrefix1.png
    - **imagePrefix2.png.P** camera file for imagePrefix2.png
    - ...
  - **blockNum1__imagePrefix1** depth and normal map for imagePrefix1.png
    - **disp.dmb** depth map
    - **normals.dmb** normal map
  - **blockNum1__imagePrefix2** depth and normal map for imagePrefix2.png
  - ...

 > Colmap's projection matrix is [R|t], while fusibile's projection matrix is K[R|t].


## Structure from Motion

The SFM part is mainly based on COLMAP(https://colmap.github.io/).
So it is natural to use COLMAP's sfm data structure.


- **SFM(colmap)**
  - **FeatureExtraction**
  - **FeatureMatching**
  - **IncreamentalSFM**
- Stereo
  - **PatchMatch**
    - **Undistortion(colmap)**
    - **PatchMatch(colmap)**
  - Semi-Global Matching
    - **Undistortion(colmap)**
    - SGM(**aliceVision**)
- Fusion(**Fusibile**)
- Meshing(**PoissonRecon**)
- Simplyfication(**VCG** or **CGAL**)
- Texturing(**TexRecon**)

### Feature Extractor

COLMAP Sift-GPU for sitf extraction.

### Feature Matcher


### Increamental SFM

## Multi View Stereo

The MVS part is based on both COLMAP(https://colmap.github.io/) for Patch Match and AliceVision(https://github.com/alicevision/AliceVision) for Semi-Global Match.

### Undistorter

The MVS moduel uses COLMAP for undistortion.

### Depth Estimator

We use both Patch Match (PM in COLMAP) and Semi-Global Match (SGM in AliceVision).

Original PM and SGM algorithm can produce bad results at texture-less ares.

The mainly method to get rid of this is multi-scale matching.

The out puts are formatted as fusibile's input.

AliceVision creates very bad depth estimation at complex scenes. We can't make it very clearly now, but it has something to do with the multi-view occlusion.

If you want to improve the behavior of the AliceVision's algorithm, you can work on the multi-view SGM algorithm with pixel-wise view-selection.

Following papers you may refer:

> Christoph Strecha，Rik Fransens，Luc Van Gool, Combined Depth and Outlier Estimation in Multi-View Stereo

> Embedded real-time stereo estimation via Semi-Global Matching on the GPU

> On Building an Accurate Stereo Matching System on Graphics Hardware

> Multi-View Stereo with Asymmetric Checkerboard Propagationand Multi-Hypothesis Joint View Selection

> Multi-Scale Geometric Consistency Guided Multi-View Stereo (ACMM)




### Fusion

> S. Galliani, K. Lasinger and K. Schindler, Massively Parallel Multiview Stereopsis by Surface Normal Diffusion, ICCV 2015

Fisibile(https://github.com/kysucix/fusibile) is a fast gpu-based fusion method. This project uses it to fuse local depth maps into blocks of point clouds.

Fusibile need images with same size. Now we test it in same size.

However, the original fusibile version only generate the gray scale version. We use the modified version from YoYo(https://github.com/YoYo000/fusibile). This lib makes it possible for colored output.

#### Bugs

- On windows fusibile doesn't produce colored point cloud.
- Fusibile now produces very sparse point cloud. Parameters should be set more properly.
- The whole model should be separated into parts to control memory cost.

### Dense Point cloud

As the bad consistency of original point cloud, we use poisson-recon for better consistency.

We use poisson-recon's output as the dense point cloud.

The PoissonRecon on http://www.cs.jhu.edu/~misha/Code/PoissonRecon has sonme bugs.
We use COLMAP's fixed version.

### Mesh Simplification

Yu Sifan is trying to simplify mesh using VGC or CGAL.

### Texture

> Let There Be Color! --- Large-Scale Texturing of 3D Reconstructions

TexRecon is used for Texturing. We use an old version(https://github.com/andre-schulz/mvs-texturing/tree/cmake). Only this version can be build successfully on Windows.

# Sample Code

```cpp
std::string project_path = "H:/RPFTest/";
std::string image_path = "H:/RPFTest/images";

Sgg4DRecon::FeatureExtractorOptions fe_ops(project_path, image_path,
Sgg4DRecon::CameraModel::SimpleRadial, true);
Sgg4DRecon::FeatureExtractor fe(fe_ops);
fe.Start();
fe.Wait();

Sgg4DRecon::FeatureMatcherOptions fmOptions(project_path);
Sgg4DRecon::FeatureMatcher fm(fmOptions);

fm.Start();
fm.Wait();

Sgg4DRecon::IncreamentalMapperOptions imOptions(project_path, image_path);
Sgg4DRecon::IncreamentalMapper im(imOptions);

im.Start();
im.Wait();

Sgg4DRecon::UndistorterOptions unOptions(project_path, image_path, 0, Sgg4DRecon::Quality::Low);
Sgg4DRecon::Undistorter un(unOptions);

un.Start();
un.Wait();

Sgg4DRecon::PMDenseEstimatorOptions pmOptins(project_path, image_path, 0, Sgg4DRecon::Quality::Low, true);
pmOptins.pass_patch_match = false;
Sgg4DRecon::PMDepthEstimator pmde(pmOptins);

pmde.Start();
pmde.Wait();

Sgg4DRecon::FuserOptions fuserOptions(project_path, 0, Sgg4DRecon::FuserType::Colmap, Sgg4DRecon::Quality::Low);
Sgg4DRecon::Fuser fuser(fuserOptions);

fuser.Start();
fuser.Wait();

Sgg4DRecon::MesherOptions meOptions(project_path, 0, Sgg4DRecon::Quality::Medium);
Sgg4DRecon::Mesher mesher(meOptions);

mesher.Start();
mesher.Wait();

Sgg4DRecon::TexturerOptions teOptions(project_path, 0);
Sgg4DRecon::Texturer texturer(teOptions);

texturer.Start();
texturer.Wait();

Sgg4DRecon::SGMDenseEstimatorOptions sgmOptions(project_path, 0);
Sgg4DRecon::SGMDepthEstimator sgmEstimator(sgmOptions);

sgmEstimator.Start();
sgmEstimator.Wait();

Sgg4DRecon::FuserOptions fuserOptions(project_path, 0);
Sgg4DRecon::Fuser fuser(fuserOptions);

fuser.Start();
fuser.Wait();
```

# Acknowlegments

This note is co-wrote by:

Co-Writer | Web Site
--|--
Yu Sifan | https://github.com/ysf1996
