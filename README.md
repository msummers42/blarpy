# blarpy
Automatically exported from code.google.com/p/blarpy

The Skinny
Exploring data to find unusual content? Blarpy is for you.

It's fast: A single thread on a typical Linux log file is ~10K lines/sec.

It's memory efficient: Blarpy can operate on enormous datasets using only a single thread on a typical computer. ex. 100GB, 300M vectors, required ~ 1GB of RAM for full statistical analysis

It's disk efficient: output is typically 1,000x-10,000x smaller than source data, depending on block size. ex. 100GB, 300M vectors, output genome was 167MB uncompressed

It's simple: "python blar.py -i filename -C" and go to work!

Blarpy makes extensive use of num.py, and the excellent bitarray module from Ilan Schnell: https://github.com/ilanschnell/bitarray

Details
Blarpy is a tool for finding anomalies in sequences, such as text or lists of vectors. It converts data to a Denoising Genomic Sequence (DGS), and then analyzes it for unusual patterns. In particular, the method preserves the geometric properties of the sequence, enabling probabilistic analysis in a tiny fraction of the space.

Blocks of data (e.g. numeric vectors, lines, paragraphs, files, etc.) are converted to a genome using Rough Locality Sensitive Hashing (RLSH), which is then analyzed like DNA. RLSH stores only a few bits per block, and yet captures the geometric information in the source data. RLSH is simultaneously a clustering and a projection, and the sparseness of high-dimensional space enables good cluster separation. In this way, Blarpy benefits from, rather than being hindered by, the curse of dimensionality.

Blarpy genomes are often 1,000x to 10,000x smaller before compression than the source data. The output genome is a stream of bits, using <= 8 bits per block. Because similar blocks are clustered probabilistically to the same binary code, the output is extremely compressible, depending on the self-similarity of the source data. DGSs are lossy sequence compression and so are an instance of Shannon's rate distortion theory for arbitrary binary data.

FAQ
How does RLSH compare to SDHash or Context Triggered Piecewise Hashing (CTPH) frequently used to compare malware files via the ssdeep signature?: Philosophically, they are similar. Analysis is in progress in prep for release v0.2.

What's the best compressor to use for storing these?: In extensive testing, Matt Mahoney's wonderful zpaq -method 3 results in the best compression by a factor of 2 over the next best compressor (bzip2). In general, BWT and Run Length Encoding compressors work best due to the large runs of repetitive bits. As always, the more random your dataset, the less compressible it is.

Are there STDIO tools so I can use tail to generate archive genomes?: You bet. csv2lsh converts numeric csv files into RLSHs. Additionally, vectext vectorizes text files into numeric csv using feature hashing and vectorization schemes, if you are analyzing text.

What's the workflow?: Blarpy is intended to be a standalone workspace, and is approximately twice as fast as the STDIO versions due to their IPC overhead. Additionally, Blarpy autotunes various parameters based on file statistics which you must set with the command line tools. The most common workflow is for analysts to first investigate their dataset using Blarpy and then automate use cases with the STDIO tools. Archived genomes can be investigated with Blarpy without suffering the re-encoding cost.

Who cares about storage space?: Remember that Blarpy's binary representation may be directly used for distance measurement, and thus does not have to be transformed first. For example, 1,000,000 128-D vectors with 64bit integers (~1GB of memory) would only require 610K of memory using RLSH's, with each vector taking only 5 bits. That's 1638 vectors in the space of 1 normal one.

Why did you choose xxHash?: https://code.google.com/p/xxhash/ It was the fastest hash in extensive testing, slightly faster than City Hash in this use case, and roughly 80% faster than MurmurHash3. It should be noted that there is considerable variability in the performance of the various Python implementations of xxHash, with pyhashxx being more than twice as fast as the next best. Why not choose City Hash? To promote diversity.

Deep FAQ
Is an RLSH just a truncated LSH?: No. There are various constructions for RLSHs from a source LSH, such as using a bit sampling LSH to reduce the source LSH, or picking columns that have the most uniform distribution. Blarpy simply uses a smaller number of bits for speed. In practice, this works well enough for most applications.

What is a Lyndon wavelet?: A Lyndon wavelet is a discrete wavelet of even length (a sequence with an equal number of each value, e.g. 50/50 0's and 1's for binary) from the set of all Lyndon words up to the size of the LSH. Lyndon wavelets are non-linear (e.g. 2min(x, 1-x)). The set of all Lyndon wavelets is equivalent to the set of all symmetric periods in the tent map, the simplest discrete-time dynamical system.

You're using projections, why do you call them wavelets?: Because they are the same. The Walsh-Hadamard transform is the binary linear wavelet decomposition, or alternately, the binary Fourier transform, and generates linearly patterned bitmasks of wavelets of a period at most equal to the LSH size. The Daubechies transform is the binary non-linear wavelet decomposition, and generates non-linear patterned bitmasks. "Random projections", "random sparse projections", and "very random sparse projections" in the definition of Achlioptas, and later Ping Li, generate random wavelets. These wavelets are not necessarily of a period at most equal to the LSH size due to their probabilistic definition. This is equivalent to an injective mapping to the set of all possible random wavelets of arbitrary period containing that 'snippet'.

There is no way a projection this low dimensional can be useful: There seem to be two reasons why RLSH works. First, for any sufficiently complex alphabet (e.g. chars), with blocks of enough length (more than 8), the dimensionality of the space makes "real world" data extremely sparse relative to all possible values. This means that although the RLSH is extremely rough, it still captures enough information, with enough separation, to be useful. Second, the predictable error qualities of the RLSH are ultimately governed by the Johnson-Lindenstrauss lemma, and you are using the error to cluster similar vectors for free. Awww snap!

What about dense patterns?: Up the voltage (to quote Jerry Hathaway). To increase resolution using autotuning, set a higher level of granularity (default: 1). To increase resolution manually, increase the feature vector length, and the alphabet width. This is equivalent to reducing the volume of the cluster region represented by the RLSH.

How do I use these RLSH's outside of blarpy?: To estimate the distance, find the Hamming distance between two blarpy strings ( count(A XOR B) ). Not only are these calculations individually faster than normal ones, many more vectors may be held in cache, accelerating pairwise comparisons over big data sets.

Where are the papers?: This is applied research with deep theoretical validation. The introduction of Blarpy accompanies a number of new data science methods, namely Denoising Genomic Sequences, Rough Locality Sensitive Hashing, and moving geometric median smoothing. These were released for the first time at the DEF CON 22 SkyTalks by Rob Bird, 8/2014. You may cite the presentation linked on the sidebar.

Are Denoising Genomic Sequences the first useful, working example of a lossy general-purpose compressor?: Possibly, depending on your definition. If you mean lossy for a universal model, again, possibly. Something as simple as gzip without keeping all characters could be considered lossy compression, though its usefulness is limited. Some papers allude to other algorithms for lossy sequence compression, but there are no working examples in the wild (that I could find...I'd love to see and use them!). An optimal central sequence in the delta compression of multiple versions of a file could be viewed as a form of lossy compression if you drop the deltas. Colin Percival discusses the idea of "universal" delta compression (his quotes) in chapter 3 of his thesis. Geometric coding also approaches the idea of compressing geometrically similar components.
