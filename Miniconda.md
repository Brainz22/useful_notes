## Installing Miniconda on MacOS:

1. Find the link to download miniconda. Then, copy the link for "bash" that best suits your MacOS. I found miniconda at https://docs.conda.io/en/latest/miniconda.html and copied the link: https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-x86_64.sh. 

2. You need make sure you can use the command `wget`. If not, you can install it using brew:
<pre>
brew install wget
</pre>

3. Once that is done, use `wget` on the link for "bash" you copied on step 1. Basically, 
<pre>
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-x86_64.sh
</pre>
This will download a `.sh` file.

4. Run the file as follows:
<pre>
bash Miniconda3-latest-MacOSX-x86_64.sh
</pre>

5. Accept all of the steps. Test conda by running:
<pre>
conda list
</pre>

This should show you a list of packages installed by the miniconda installation, and this means that you can now use `conda`.
