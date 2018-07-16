# masterHI

masterHI is an semi-automatic script generator for the processing of **Bruker formatted** non-uniformally sampled **3-dimensional** data by the **hmsIST program**. We exploit the **nmrPipe** data format and Fourier Transform functions to aid in this process.

The basic workflow of non-uniformally sampled data reconstruction is:

* conversion from Bruker format to nmrPipe format

* a Fourier transform of the direct dimension first along with phase correction

* a reconstruction step of all the indirect 2D planes for each point in the processed direct dimension

* a final Fourier transform of the indirect 2D planes

These four steps are carried out by hmsIST by using seven independent scripts that perform each task along the processing pipeline - masterHI will create these scripts for you. They are called (and do):

* **convert.com** (conversion from Bruker to nmrPipe format)

* **phase.com** (Fourier transforms and phases direct dimension) and **prepare4recon**.com (prepares data for reconstruction)

* **recon.py** and **hmsist.com** (parallelizes instances of hmsist.com for reconstruction) and **prepare4ft.com** (reformats the data)

* **ft.com** (performs the final Fourier transformation of the 2 indirect dimensions)


## The Good News

The good news is masterHI generates and executes these scripts. You interact with masterHI on the command line by using command line arguments. These arguments are stored between steps so they only need to be given once unless you want to change them later during processing or reprocessing. Many of these options are gathered from the data directory so are automatically acquired for you and don't need to be provided as command line arguments. You can always override these options with command line arguments.

## Example processing (lets get started quickly version)

### Step 1: Conversion

Convert the data from Bruker to nmrPipe format with:

```
> masterHI --conv
```

This will fail if the following files are not in the data directory:

* nuslist
* ser
* acqus
* acqu2s
* acqu3s


### Step 2: Fourier Transform of direct dimension and phase check

The first dimension is directly acquired and can be processed with a regular FT - but we must correct for the phase. To do so, execute:

```
> masterHI --phasecheck
```

This takes the first 4 FIDs (the first sampled point), does an FT on them and loads the result into nmrDraw. In nmrDraw you can usually find one or two FIDs out of these four with good signal and correct the phase as you would normally do so in nmrDraw. Not the phase correction. Lets say it is 39.2 for phase 0 (p0) and no phase correction for phase 1 (p1). We check we have the phases correct by executing:

```
> masterHI --phasecheck --phase0 39.2 --phase1 0
```

Note: you can simply not add the `--phase1` argument to make things briefer.

This will redo the transformation and load the result in nmrDraw for you to view again. Make any other fine adjustments if you think they are needed and repeat the above command. Keep in mind that if you found you needed, say, another 5 degrees correction, you should use `--phase0 44.2`, not `--phase0 5.0`. Always check that you are happy with the phase before moving on. You can simply run:

```
> masterHI --phasecheck
```

as a final check.

### Step 3: Reconstruction

Reconstruction can take many command line variables but mostly what you will want is default or can be detected from the data directory. The simplest thing to do, which will work in probably about 95% of cases is:

```
> masterHI --recon
```

Briefly, this will FT and phase correct all the data transpose it to the indirect dimensions are ready for reconstruction. It will then start the reconstruction. It will automatically detect how many cpus/threads it can use and will use 100% of the resources it can. TO limit the number of concurrent processes, use the `--proc` argument:

```
> masterHI --recon --proc 2
```
This will only use two concurrent processes.

You should see an output that looks something like this:

```
Preparing Sampled Points for Full Reconstruction (prepare4recon.com)
FT       412 of 412    H     
Performing Reconstruction (recon.py / hmsist.com)
[####----------------------------] 15.33% done
```



## Detailed Example
### Step 1: Conversion

We begin by converting the data from Bruker format to nmrPipe format. Bruker raw data is stored in the 'ser' file within the Bruker data directory, but in order to get frequency referencing information and certain other details, masterHI looks for acqus, acqu2s, acqu3s, puleprogram and nuslist files as well. These files must all exist in the data directory for masterHI to proceed. They should however, all be there. So to convert, move to the data directory and run:

```
> masterHI --conv
```

N.B. All command line options are given with double-dashes ('--').

This creates a file called 'fid.com' and executes it.

Alternatively you can run the script from any directory to keep the processing out of the data directory. For example its common to create a processing directory below the data directory and work in there. But you must indicate where the data directory is. For example:

```
> mkdir PROC
> cd PROC
> masterHI --conv --dir ..
```
#### Option: Controlling number of samples requested

Sometimes you might want to use less samples than
