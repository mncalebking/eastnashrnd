---
published: true
layout: post
title:  "High-speed Password Hashing with Google Cloud and Hashcat"
author: caleb
categories: [ caleb, tutorial ]
image: assets/images/6.jpg
---

# Hashing passwords with Google Cloud and Hashcat

1. Create an accunt with Google Cloud Services.

2. Enable billing. 

3. Under quotas, select GPUs (all regions) and NVIDIA P100 GPUs for whichever region you plan on using. I chose us-west1. Then click "Edit Quotas." Enter your information, then enter a new quota limit of 1-4. You will also need to enter your reason for requesting a higher quota. The quota request process usually takes around 24 hours. 

At the time of writing this guide, a VM with 4 Nvidia Tesla GPU's costs about $4/hour. 

4. Create a new VM with the following settings:
Compute engine > VM instances > Create instance.

Region: The region you chose for your quota request.
Machine type > customize. 2vCPUs 13GB memory. 1-4 Nvidia Tesla P100 GPUs
Boot disk: Ubuntu 18.04 LTS. SSD persistent disk. The disk size you expect to use (36gb recommended). 

No service account.

5. Connect via SSH. 

First we'll be updating the software on the server. 
{% highlight bash %}
	sudo apt-get update && sudo apt-get upgrade
{% endhighlight %}
Next we'll install hashcat and gdebi
{% highlight bash %}
	sudo apt-get install hashcat gdebi
{% endhighlight %}
Enter Y if prompted

6. Drivers
Go to https://www.nvidia.com/Download/index.aspx?lang=en-us 
Product type: Tesla
Series: P Series
Product: Tesla P100
Operating System: Linux 64-bit Ubuntu 18.04 (you may have to click "show all Operating Systems")
CUDA Toolkit: 10.0

Press search, then download. Then copy link address on the "Agree and Download" button. 

Replace <link> with the link copied from the nvidia site. 
{% highlight bash %}
wget <link>
{% endhighlight %}

Example:
{% highlight bash %}
me@research-1:~$ wget http://us.download.nvidia.com/tesla/410.79/nvidia-diag-driver-local-repo-ubuntu1804-410.79_1.0-1_amd64.deb
--2019-02-07 14:36:58--  http://us.download.nvidia.com/tesla/410.79/nvidia-diag-driver-local-repo-ubuntu1804-410.79_1.0-1_amd64.deb
Resolving us.download.nvidia.com (us.download.nvidia.com)... 192.229.211.70, 2606:2800:21f:3aa:dcf:37b:1ed6:1fb
Connecting to us.download.nvidia.com (us.download.nvidia.com)|192.229.211.70|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 97061646 (93M) [application/octet-stream]
Saving to: ‘nvidia-diag-driver-local-repo-ubuntu1804-410.79_1.0-1_amd64.deb’

nvidia-diag-driver-loca 100%[============================>]  92.56M   338MB/s    in 0.3s    

2019-02-07 14:36:58 (338 MB/s) - ‘nvidia-diag-driver-local-repo-ubuntu1804-410.79_1.0-1_amd64.deb’ saved [97061646/97061646]
{% endhighlight %}

Install Cuda:
{% highlight bash %}
me@research-1:~$ sudo gdebi cuda-repo-ubuntu1804-10-0-local-10.0.130-410.48_1.0-1_amd64
Reading package lists... Done
Building dependency tree        
Reading state information... Done
Reading state information... Done

cuda repository configuration files
 Contains repository configuration for cuda.
 Contains a local repository for cuda.
Do you want to install the software package? [y/N]:y
{% endhighlight %}

Add the repo key:
{% highlight bash %}
sudo apt-key add <path to key>.pub
{% endhighlight %}
  
Repeat the process for the nvidia-diag-driver-local-repo-ubuntu1804-410.79_1.0-1_amd64.deb file

Then update the package manager and install cuda:
{% highlight bash %}
	sudo apt-get update && sudo apt-get install cuda
{% endhighlight %}
  
Once completed, reboot the server.
{% highlight bash %}
	sudo reboot
{% endhighlight %}
  
Reconnect to the instance and verify that the drivers have been successfully installed:
{% highlight bash %}
	hashcat -I
{ % endhighlight %}

You should see something like this:

{% highlight bash %}
	me@research-1:~$ hashcat -I
	hashcat (v4.0.1) starting...

	* Device #1: This device's constant buffer size is too small.

	* Device #1: This device's local mem size is too small.

	OpenCL Info:

	Platform ID #1
	  Vendor  : The pocl project
	  Name    : Portable Computing Language
	  Version : OpenCL 1.2 pocl 1.1 None+Asserts, LLVM 6.0.0, SPIR, SLEEF, DISTRO, POCL_DEBUG

	  Device ID #1
		Type           : CPU
		Vendor ID      : 128
		Vendor         : GenuineIntel
		Name           : pthread-Intel(R) Xeon(R) CPU @ 2.20GHz
		Version        : OpenCL 1.2 pocl HSTR: pthread-x86_64-pc-linux-gnu-broadwell
		Processor(s)   : 2
		Clock          : 2200
		Memory         : 4096/10969 MB allocatable
		OpenCL Version : OpenCL C 1.2 pocl
		Driver Version : 1.1

	Platform ID #2
	  Vendor  : NVIDIA Corporation
	  Name    : NVIDIA CUDA
	  Version : OpenCL 1.2 CUDA 10.0.211
	  
	  Device ID #2
		Type           : GPU
		Vendor ID      : 32
		Vendor         : NVIDIA Corporation
		Name           : Tesla P100-PCIE-16GB
		Version        : OpenCL 1.2 CUDA
		Processor(s)   : 56
		Clock          : 1328
		Memory         : 4070/16280 MB allocatable
		OpenCL Version : OpenCL C 1.2 
		Driver Version : 410.79

	  Device ID #2
		Type           : GPU
		Vendor ID      : 32
		Vendor         : NVIDIA Corporation
		Name           : Tesla P100-PCIE-16GB
		Version        : OpenCL 1.2 CUDA
	
	  Device ID #3
		Type           : GPU
		Vendor ID      : 32
		Vendor         : NVIDIA Corporation
		Name           : Tesla P100-PCIE-16GB
		Version        : OpenCL 1.2 CUDA
		Processor(s)   : 56
		Clock          : 1328
		Memory         : 4070/16280 MB allocatable
		OpenCL Version : OpenCL C 1.2 
		Driver Version : 410.79

	  Device ID #4
		Type           : GPU
		Vendor ID      : 32
		Vendor         : NVIDIA Corporation
		Name           : Tesla P100-PCIE-16GB
		Version        : OpenCL 1.2 CUDA
		Processor(s)   : 56
		Clock          : 1328
		Memory         : 4070/16280 MB allocatable
		OpenCL Version : OpenCL C 1.2 
		Driver Version : 410.79

	  Device ID #5
		Type           : GPU
		Vendor ID      : 32
		Vendor         : NVIDIA Corporation
		Name           : Tesla P100-PCIE-16GB
		Version        : OpenCL 1.2 CUDA
		Processor(s)   : 56
		Clock          : 1328
		Memory         : 4070/16280 MB allocatable
		OpenCL Version : OpenCL C 1.2 
		Driver Version : 410.79
{% endhighlight %}
  
## Create an MD5 hash of input text
{% highlight bash %}
	echo -n "test" | md5sum | tr -d " -" > hash
{% endhighlight %}
Outputs to a file called "hash"

## Hashcat Masks
Masks allow for drastically improved performance by lowering the number of combinations to be tested. 

Lets say, for example, that the password we want to crack uses the following rules:
  * 6-12 characters
  * Uppercase and lowercase letters
  * Numbers 0-9

{% highlight bash %}
	-1 ?u?l?d	
{% endhighlight %}
  
This character set includes uppercase letters, lowercase letters, and digits.
{% highlight bash %}
hashcat -a 3 -1 ?u?l?d hash ?1?1?1?1?1?1?1?1?1?1?1?1
{% endhighlight %}
Using this command, it'll take about 602275007914781 days or 1650068514835 years to hash every possible combination. Obviously this isn't going to work. Instead we'll just hash up to 7 characters. If the target password is longer than this, we'll have to resort to dictionary-based attacks. 
  
{% highlight bash %}
hashcat -a 3 -1 ?u?l?d hash ?1?1?1?1?1?1?1
{% endhighlight %}
This command will only work for 7 character passwords. If we use the --increment option, hashcat will hash all possible combinations between 1-7 characters. Hashing passwords under 6 characters is completely pointless, so you may want to use a mask file.

### Mask Files
Using a .hcmask file allows the user more flexibility than the standard hashcat options. For example, if we want to try all possible 6-7 character combinations, we can use a mask file like this:
{% highlight bash %}
?1?1?1?1?1?1
?1?1?1?1?1?1?1
{% endhighlight %}
  
To use the mask file, place it on the line where you'd usually put the mask. 
{% highlight bash %}
hashcat -a 3 -1 ?u?l?d file.hash example.hcmask
{% endhighlight %}


---

{% highlight bash %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
