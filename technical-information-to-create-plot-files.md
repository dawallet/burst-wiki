Introduction
============

The term plotting is a name for dedicating storage space to be used for calculations in the Burstcoin network. A Plot is a file containing pre-computed hashes that can be used to forge blocks for Burstcoin blockchain. The plots are later used by mining software and can be thought of as the miner’s hash rate.

Algorithms and acronyms
=======================

Before we dive deep into how plotting works we need to get familiar with all different terms used in the procedure.

Shabal  
Shabal is the name of the crypto/hash function used in Burstcoin. Shabal is a rather heavy and slow crypto in relation to many other like i.e. SHA256. Because of this it makes it a good crypto for Proof of capacity coins like Burstcoin. This is because we store the precomputed hashes while it is still fast enough to do smaller live verifications. Burst uses the 256bit version of Shabal also known as Shabal256.

<!-- -->

Hash / Digest  
A hash or digest in this context is a 32Byte (256bit) long result of the Shabal256 Crypto.

<!-- -->

Nonce  
When generating a plot file, you generate something that is called nonces. Each nonce contains 256Kilobyte of data that can be used by miners to calculate Deadlines. Each nonce will have its own individual number. This number can range between 0-18446744073709551615. the number is also used as a seed when creating the nonce. Because of this each nonce has its own unique set of data. One plot file can contain many nonces.

<!-- -->

Scoop  
Each nonce is sorted into 4096 different places of data. These places are called scoop numbers. Each scoop contains 64byte of data which holds 2 hashes. Each of these hashes are xored with a final hash (we get to final hash in generating a nonce chapter).

<!-- -->

Account ID  
When you create your plot file it will be bound to a specific Burst account. The numeric account ID is used when you create your nonces. Because of this all miners have different plot files even if they use the same nonce numbers.

Generating a nonce
==================

The first step in creating a nonce is to make the first seed. The seed is a 16byte long value containing the account id that we will be generating a nonce for and the nonce number. When this is done we start to feed the Shabal256 function to get our first hash.

![](PlottingStep1.png "PlottingStep1.png")

We have produced the first hash. This is the last hash in the nonce. Hash \#8191. Now we take this produced hash (\#8191) and pre-append it to the starting seed. The result will now be our new seed for the next round of shabal256 computation.

![](Plottingstep2.png "Plottingstep2.png")

We now have produced two hashes. Hash \#8191 and Hash \#8190. This time we pre-append Hash 8190 to the last seed we used. The result will now be a new seed to feed Shabal256.

![](Plottingstep3.png "Plottingstep3.png")

Once again, we have created a new hash. This procedure of pre-appending resulting hashes to a new seed will continue for all 8192 hashes we create for a nonce. After iteration 128 we have reached more than 4096 bytes in the seed. For all remaining iterations we will only read the last 4096 generated bytes.

![](Plottinglastgenerated.png "Plottinglastgenerated.png")

Once we have created 8192 hashes we are now going to make a Final hash. This is done by using all 8192 hashes and the first 16bytes as seed.

![](Plottingfinal.png "Plottingfinal.png")

The final hash will now be used to xor all other hashes individually.

![](Plottingxor.png "Plottingxor.png")

We have now created our nonce and can store it in a plot file before we continue to the next nonce.

![](Plottingnonce.png "Plottingnonce.png")

Plot structure
==============

When we are mining we read from one or more plot files. The miner software will open a plot file and seek the scoop locations to read the scoops data. If the plot file is unoptimized the scoop locations will be on more than one place. In the following example the miner will be seeking and reading scoop \#403.

![](plottingunoptimized.png "plottingunoptimized.png")

This is not the most effective way since the miner will spend a lot of time to seek new locations on the storage device to be able to read the scoops. To prevent this, we can optimize plots or use plotter software that creates optimized plots from the beginning. Optimization is done by reordering the data in the plot file and grouping all data from the same scoop number together.

![](Plottingoptimization.png "Plottingoptimization.png")

Basically, what we have done is to divide the plot file into 4096 portions where we split up all the nonces data based on scoop numbers. When the miner now wants to read Scoop 4096 it only seeks one time and read all data sequentially. This provides better performance.

Stagger and filenames
=====================

Stagger  
A stagger is basically a group of nonces in a plot file. The groups in the plot file is written in an optimized way. A given stagger number tells you how many nonces there are in each group. To find out how many groups there are in a plot file you take the number of nonces and divide it with the stagger number. If the stagger number is equal to the number of nonces in the file, there is only one group and the plot file is completely optimized. If this is the case the miner will not care about the stagger. If for some reason your division ends up with decimals, your plot file can be assumed broken.

<!-- -->

Filenames  
Since a plot file only contains raw data there is no headers in the files. All information needed for a user and miner is set in the filenames. The formatting of the filename is as follows.

    AccountID_StartingNonce_NrOfNonces_Stagger

------------------------------------------------------------------------

This information is written by community member @Quibus 2017-10-13