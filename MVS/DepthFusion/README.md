# Depth Fusion

There're mainly tow kinds of ways to fuse local depth map into a global point cloud.

In **SFMVS**, we use fusibile for GPU-based fusion.


> S. Galliani, K. Lasinger, and K. Schindler, “Massively Parallel Multiview Stereopsis by Surface Normal Diffusion,” in 2015 IEEE International Conference on Computer Vision (ICCV), Santiago, Chile, 2015, pp. 873–881.

> S. Donne and A. Geiger, “Learning Non-volumetric Depth Fusion using Successive Reprojections,” p. 10.

## Volumetric Fusion

initially proposed by Curless et al. [3], wasmadeincreasinglypopularbyZachetal.[39]andKinectFusion [26], fusing various depth maps into a single truncated signed distance ﬁeld (TSDF). Leroy et al. have recently integrated this TSDF-based fusion into an end-to-end pipeline[22]. Fuhrmannetal.discusshowtohandlethecase of varying viewing distances and at the same time do away with the volumetric grid in favour of a point list [5] which scales better. Other non-learning-based techniques have been proposed to counter this scaling behaviour, such as a hybrid Delaunay-volumetric approach [25] and octrees [34]. The ﬁrst does not lean itself well for learning-based approaches, but three concurrent works have leveraged hierarchical surface representations (i.e., octrees) to improve execution speed [9,29,32]. However, even such approaches have issues scaling beyond 5123 voxels: eventually, they hit a computational ceiling. By working in the image domain, we largely sidestep the scaling issue and can additionally lean on the large amount of work and understanding available for image-based deep learning.

## Image-based Fusion
 promises quadratic rather than cubic scaling. Traditionally, it only discards reconstructed points that are not supported by multiple views. This is implemented in Gipuma as the Fusibile algorithm [8], and Xu et al. [36] use the same fusion technique in AMHMVS, their learning-based version of Gipuma. In COLMAP [30], the accepted pixels are clustered in “consistent pixel clusters” that are combined into a single reconstructed point cloud: clusters not supported by a minimum number of views are discarded. Similarly, Poggi et al. [28] and Tosi et al. [33] have leveraged deep learning to yield conﬁdence estimates. Whiletheformertechniquesﬁlteroutbaddepthestimates, they do not attempt to improve the estimates. We argue that the depth maps can still be signiﬁcantly improved by incorporating information from neighbouring views. To the best of our knowledge, learning-based reﬁnement of depth maps was only done in a single-view setting [15,37]. We aim to learn depth map fusion and reﬁnement from a variable number of input views – the zero-neighbour variant of our approach serves as the single-image baseline similar to [15] and [37]. Our combined approach notably improves the quality of the fused point clouds, quantiﬁed in terms of the Chamfer distance (see Section 5), and at the same time yields improved depth maps for all input views.
