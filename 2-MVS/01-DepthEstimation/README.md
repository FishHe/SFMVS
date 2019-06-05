# ACMH

>  Xu Q , Tao W . Multi-View Stereo with Asymmetric Checkerboard Propagation and Multi-Hypothesis Joint View Selection[J]. 2018.

>  Galliani S , Lasinger K , Schindler K . Massively Parallel Multiview Stereopsis by Surface Normal Diffusion[C]// 2015 IEEE International Conference on Computer Vision (ICCV). IEEE Computer Society, 2015.

Now I'm trying to repeat the ACMH algorithm with cuda.

I refer to Gipuma(https://github.com/kysucix/gipuma).

The cuda structure is defined as follows:

![](assets/markdown-img-paste-20190604213350699.png)

* **Grid** The Block container.
* **Block** The Thread container.
* **Thread** The GPU Thread.
* **Tile** The data area that is managed by the Block.
* **Shared Area** The shared area related to a certain tile and its expansion.

The tile_size is defined with both block_size and expansion_size.

The checkerboard algorithm will be implemented in this way.

# ACMM

ACMM is multi-scale version of ACMH. It works better in texture-less areas.

> Q. Xu and W. Tao, “Multi-Scale Geometric Consistency Guided Multi-View Stereo,” arXiv:1904.08103 [cs], Apr. 2019.
