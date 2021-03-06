---
title: "Making Changes to an Existing Code"
teaching: 20
exercises: 60
questions:
- "How do I make changes to a code and then push it to GitHub?"
objectives:
- "Use the module documentation to identify new features that can be used to analyze data."
- "Save code as a command line python script."
keypoints:
- "GitHub maintains text-based files better than more complex file formats like jupyter notebook files."
---
## Getting ready
- On Windows: Make sure that you are using the “Anaconda Powershell Prompt” which will give you access to both the git and python commands.  [For our purposes Powershell should allow you to follow the macOS/Linux-based instructions in this exercise.  There may be occasional exceptions.]

- On Mac: All commands will work normally in the Terminal.

## Overview

In this activity, you will:
- Fork a GitHub repository (containing a Python script that runs Psi4) to your personal account.
- Clone the repository to your local computer.
- Understand the Psi4 script so that you will feel comfortable modifying it.
- Modify the Psi4 script so that it can do something new (save off vibrational frequencies and degeneracies), making use of several NumPy functions.
- Commit your changes and push them to your personal repository on GitHub.
- Create a pull request for your features to be added back to the original repository.

## Fork the original GitHub repository to your personal account.
Log into your GitHub account, visit the original repository located at https://github.com/pnerenberg/molssi-qm-synthesis and click the “Fork” button on the upper right.

This will make a copy of the original repository under your personal account.  It contains all of the code in the original repository; the only difference is that it knows it’s derived from the original one.

Your forked repository is located at https://github.com/[your_username]/molssi-qm-synthesis

[Note: You will be able to make changes to your forked repository, but not the original repository.  This is a key aspect of how GitHub manages multiple people contributing features to the same code.]

## Clone the repository to your computer.
Make sure you’re in the molssi-day2 folder on your Desktop.  Then run the command:
```
git clone https://github.com/[your_username]/molssi-qm-synthesis.git
```

This will create a folder called molssi-qm-synthesis in the folder where you ran the command.

Enter the folder and you will see two files: Methane-VibFreqAnalysis.py and README.

## Understand the script.

Methane-VibFreqAnalysis.py is a Python script that contains (surprise, surprise!) the beginnings of a vibrational frequency analysis of methane.  In a change from Activity #4, the frequencies are accessed directly using the frequency_analyis method of the wavefunction object.  In the script you’ll see a line containing the following code:
```
print(scf_wfn.frequency_analysis)
```
This statement causes all of the info generated by the frequency_analyis method to be output to the screen.  Changing the code here will be the “departure point” for this activity.

Up until this point, we’ve been doing a lot of coding in jupyter notebooks.  They have the advantage that you can write code interactively, but sometimes we want to make “traditional” scripts or programs that run from start to finish.  Here you have two options for running the code:
- Option #1
From a command prompt inside the folder, run the script using the command:
python Methane-VibFreqAnalysis.py

The script will run and the print command mentioned above will result in the vibrational frequency information being printed to the screen.

- Option #2
You may paste the script into the notebook and run it.  It’s best to organize things as:

Cell 1: Import statements
Cell 2: Basic setup (e.g., molecular geometry)
Cell 3: The rest of the code (running the calculation)

When you execute the cells the vibrational frequency information will be displayed in the notebook.

## Modifying the Script

You will modify the Psi4 script to carry out the vibrational frequency analysis, extract the unique non-zero frequencies and their degeneracies, and save this information to a text file.

The cool thing about PsiAPI is that your quantum chemistry calculation is inside of the Python script, so you have a lot of flexibility to customize it.  Once again you may use the jupyter notebook to help develop your code, but ultimately we want it to be part of the Python script.

The first part of the code imports the required python modules, sets up your molecule (methane), and performs a geometry optimization calculation.
```
import psi4
import numpy as np

# Initial setup
psi4.set_memory('2 GB')
psi4.set_num_threads(2)

file_prefix = 'methane_HF-DZ'

ch4 = psi4.geometry("""
symmetry c1
0 1
   C       -0.85972        2.41258        0.00000
   H        0.21028        2.41258        0.00000
   H       -1.21638        2.69390       -0.96879
   H       -1.21639        3.11091        0.72802
   H       -1.21639        1.43293        0.24076
""")


# Geometry optimization
psi4.set_output_file(file_prefix + '_geomopt.dat', False)
psi4.set_options({'g_convergence': 'gau_tight'})
psi4.optimize('scf/cc-pVDZ', molecule=ch4)
```
{: .language-python}

The next section runs a frequency calculation.
```
# Run vibrational frequency analysis
psi4.set_output_file(file_prefix + '_vibfreq.dat', False)
scf_energy, scf_wfn = psi4.frequency('scf/cc-pVDZ', molecule=ch4, return_wfn=True, dertype='gradient')
```
{: .language-python}

This command outputs the energy to the variable `scf_energy` and saves the wavefunction as the variable `scf_wfn`.  You can now do other things using that wavefunction object.  As discussed above, first we will use the frequency analysis command to print everything associated from the frequency analysis.
```
print(scf_wfn.frequency_analysis)
```
{: .language-python}

The beginning of the output from this print statement looks like this.
```
{'q': QCAspect(lbl='normal mode', units='a0 u^1/2', data=array([[ 3.78131853e-01,  1.51117558e-04,  3.22800932e-02,  
```
{: .output}

Study the full output of this command.  The `scf_wfn.frequency_analysis` generates a dictionary containing all of the vibrational frequency information.  (Here’s a [quick refresher](https://www.learnpython.org/en/Dictionaries) about dictionaries.) The vibrational frequencies (in cm<sup>-1</sup>) are contained in the entry omega, which itself is a special type of Python object (for Psi4) called “QCAspect”.  Practically speaking, you can treat this object like a list or tuple, and you will find the frequencies as one of the entries within this object.  *Hint: Use square bracket notation!*

For the next Exercises, all of these features can be implemented using functions from the NumPy library, and we highly encourage you to make use of Google/your favorite search engine or the [NumPy documentation](https://docs.scipy.org/doc/numpy/reference/index.html) to find the necessary/useful functions.

> ## Exercise: Extract the frequencies from the dictionary to a variable
> Save the frequencies as a variable called `rawfreq`.  
>
>> ## Solution
>> As you move on to the next steps, you probably don't want to print the frequencies, but we do here for illustrative purposes.   
>> ~~~
>> rawfreq = scf_wfn.frequency_analysis['omega'][2]
>> print(rawfreq)
>> ~~~
>> {: .language-python}
>>
>> ~~~
>> [0.00000000e+00+4.46337942e-05j 0.00000000e+00+2.72283897e-05j
>>  0.00000000e+00+1.26593199e-05j 5.39970123e-06+0.00000000e+00j
>>  2.98741425e-05+0.00000000e+00j 3.78043286e-05+0.00000000e+00j
>>  1.43379876e+03+0.00000000e+00j 1.43381506e+03+0.00000000e+00j
>>  1.43382005e+03+0.00000000e+00j 1.64834185e+03+0.00000000e+00j
>>  1.64834968e+03+0.00000000e+00j 3.16483880e+03+0.00000000e+00j
>>  3.28564478e+03+0.00000000e+00j 3.28565524e+03+0.00000000e+00j
>>  3.28567217e+03+0.00000000e+00j]
>> ~~~
>> {: .output}
>> We pull out the dictionary item `omega` and then the index 2 element of that item.  (If you study the full output of the print statement above, the index 0 element of `omega` is `lbl='frequency'` and the index 1 element is the units.)
> {: .solution}
{: .challenge}

> ## Exercise: Round the real frequencies
> Eliminate imaginary parts of frequencies, round the frequencies (to the nearest whole number), and extract only the *non-zero* frequencies.  
>
>> ## Solution
>> ~~~
>> roundnzfreq = np.round(np.real(rawfreq))[6:]
>> print(roundnzfreq)
>> ~~~
>> {: .language-python}
>> ~~~
>> [1434. 1434. 1434. 1648. 1648. 3165. 3286. 3286. 3286.]
>> ~~~
>> {: .output}
>>
>> The slice `[6:]` is used because for a minimum energy structure the first six frequencies will always be zero, as they represent the translational and rotational degrees of freedom.
> {: .solution}
{: .challenge}

> ## Exercise: Determine unique frequencies and degeneracies
> Determine the unique non-zero frequencies and the number of times each such frequency occurs.  Store these in a NumPy array in the format: {frequency, count}, one line per frequency.
>
>> ## Solution
>> ~~~
>> freqlist = np.array(np.unique(roundnzfreq, return_counts=True)).T
>> print(freqlist)
>> ~~~
>> {: .language-python}
>> ~~~
>> [[1.434e+03 3.000e+00]
>>  [1.648e+03 2.000e+00]
>>  [3.165e+03 1.000e+00]
>>  [3.286e+03 3.000e+00]]
>> ~~~
>> {: .output}
>>
>> Note that `np.unique()` will generate two 1D NumPy arrays as called. Enclosing that command with np.array() creates the 2D numpy array. The `.T` is an easy way to transpose any numpy array.
> {: .solution}
{: .challenge}

> ## Exercise: Save your results to a file.
> Save the NumPy array with frequency and count data to a text file.
>
>> ## Solution
>> ~~~
>> np.savetxt('methane_FreqList.dat', freqlist, fmt='%0.1f,%d', header='freq,degen')
>> ~~~
>> {: .language-python}
>>
>> Here we've done a couple of nice things: (1) formatted the frequencies as floats with one decimal place and the degeneracies as integers; (2) included a header line so that other users will understand what's in the file.  If you look in the folder where you ran the script, you should see a file called `methane_Freqlist.dat` which contains all your data neatly formatted.
> {: .solution}
{: .challenge}

Finally, you can check your answers for the degeneracies of each vibrational frequency by looking [here.](http://www.chemtube3d.com/vibrationsch4/)

You can be creative with this exercise and consider making other modifications (e.g., to the method/basis set).

> ## IPython.embed
> Note #1: When coding in Python, there is a nifty tool that allows you to interact with a running script at any desired location in the code (e.g., after the line containing the print(scf_wfn.frequency_analysis) statement).  This is made possible by adding these two lines into the desired location:
> ~~~
> import IPython
> IPython.embed()
> ~~~
> {: .language.python}
>
> Running the script from the command prompt will pause the script after these two lines and give you an interactive prompt so that you can manipulate variables, try out new code, etc.
{: .callout}

Once you are finished with your script, you want to save it as a python script if you were working in the jupyter notebook.  This is because Git works best with codes and script, rather than more complicated file formats such as notebooks.  You can either cut and paste all your code into a text file in your favorite text editor and save it as a .py file OR you can go to File -> Download As -> Python (.py) and save your notebook as a python script.  

> ## Exercise: Commit your code to your repository GitHub
> Now that you have developed your feature, commit your changes on the local machine, and push them to your repository fork on GitHub.
>
>> ## Solution
>> The commands are:
>> ~~~
>> git commit -m “Type any commit message you want”
>> git push origin master
>> ~~~
>>
>> Here, `origin` stands for the remote repository, i.e. your repository fork on GitHub.  You can request to display the remote repository names and their URLs using the command:
>> ~~~
>> git remote -v
>> ~~~
>> Your changes are now uploaded to your repository fork on GitHub, but they are not part of the upstream repository yet.
> {: .solution}
{: .challenge}

## Create a pull request on GitHub for your code to be added to the original repository.

At this point, your code is different from that of the original repository; you may request the owner to incorporate your code changes using the pull request button.  Click the pull request button to initiate a conversation thread; the owner may accept the pull request right away (updating the original repository with your code), or ask you to make more changes.
