# Pipenv and S2i: A Better Developer Experience for Python Containers

When starting down the path to learning Python, developers are 

## The Shortcomings of Pip

`pip` is an incredibly simple tool that enables developers to easily install packages in their environment but that simplicity creates several problems that make it easy for both new and experienced developers to unknowingly introduce problems for themselves later on.

The first challenge developers often face when attempting to containerize their application is understanding exactly what packages they need to install in their container.  Developers may have installed several different tools in their environment during development and reverse engineering which packages are needed, which are dependecies, or which are left over from other projects (more on this later).  

The `requirements.txt` file is a common pattern python developers will use for tracking what packages need to be installed in their container.  It is a simple solution that at first glance appears to solve the problem, simply requiring the execution of `pip install -r requirements.txt` in the container build process.  However, several problems can still occur with a requirements.txt file.  

One issue is that developers must manually track what packages need to be added to the requirements file.  This creates a fairly minor burdon on the developer to ensure that as they install packages they record those changes in the requirements file.

Another issue with the requirements file is with tracking dependencies of packages a developer installs.  I may specify package `a` in my requirements file which then installs package `b` for me automatically.  This may work perfectly today but unbeknownst to me, I have potentially introduced a future dependency problem.  Package `a` defines the requirement for `b` as simply `b>=1.0.0` and does not specify an upper limit of the dependency version.  At some point package `b` releases an update which removes a feature that `a` is using and now my application is breaking.  

We can try and work around this by simply pinning all of the packages our application needs, dependencies and all with something like `pip freeze > requirements.txt`.  That does get all of your dependencies but you need to make sure you are freezing from a clean environment with no extra packages such or even dev tools installed.

## Introducing Pipenv, Pipfile, and Pipfile.lock

Pipenv attempts to solve many of these problems and should feel familiar to developers familiar with `npm` in the node ecosystem.  `pipenv` replaces `pip` as the tool developers use to install packages.  Unlike tools like `conda`, `pipenv` installs the same packages available from pypi.org but just replaces `pip` on your local environment.

To get `pipenv` you can just install it with `pip install pipenv`.  Once `pipenv` is installed you are ready to start installing packages.  Where you might have run `pip install requests` before you can instead run `pipenv install requests` to get the exact same package.  When running `pipenv` in a project for the first time you will immediatly see it create a file called `Pipfile`.  The `Pipfile` for our environment will look something like this:


```
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]
requests = "*"

[dev-packages]

[requires]
python_version = "3.9"
```

Just like the `requirements.txt`, the `Pipfile` is able to capture which packages we wish to install, but `pipenv` is able to automatically update it as we install packages. It also captures some other useful information, such as the Python version we are using, and information on the Pypi repository URL. Additionally it has a section for `dev-packages` which we didn't have for our `requirements.txt` file at all.  If you wish to use an automatic code formatter in your project like `black` you can simply run `pipenv install --dev black`.  This allows you to track what packages you are using for development and keep them seperate from your application requirements that are needed for deploying your final application.

`pipenv` creates another file while install applications though called `Pipfile.lock`.  The lock file handles pinning the versions of all of the packages you have installed and their dependencies.  This allows you to reinstall the exact same version of all components even if newer versions of those packages have come out since then.  If you need to rebuild your container several months down the line running `pipenv install` will install the exact package versions specified in the lock file, ensuring that changes in dependencies won't accidently break your application. `Pipfile` and `Pipfile.lock` are intended to be checked into source control so don't be intimidated by the fact that `Pipfile.lock` is automiticlly generated.

Another mistake that new Python developers often make is attempting to work from their global user Python environment.  As mentioned in the issues with `pip`, this can cause depedency confusion in your current project as well as potentially break another project that requires a specific package version.  The solution here is to utilize virtual environments.

Virtual environments allow you to create a "clean" python environment that you are able to install and manage packages independently from the global Python environment.  Python has a number of tools and methods for creating and managing virtual enviornments which can be a bit overwhelming.  

Thankfully `pipenv`, like it's name implies, will manage your environment for you.  When running `pipenv install` pipenv will automatically detect if there is already a virtual environment created for this project and either create a new virtual environment or install the packages into the existing virtual environment. That virtual environment can easily be activated with `pipenv shell` allowing you to access and run your application or packages from that virtual environment. 

```
Tip: I prefer to keep my virtual environment in my project folder with my Pipfile and by default pipenv generates it in a centrally located folder.  You can change this behavior by setting the following in your .bashrc file:

export PIPENV_VENV_IN_PROJECT=1

With this option set, pipenv will create a .venv/ folder to manage the virtual environment directly in your project folder.  This folder can easily be deleted if you want to rebuild it from scratch or you just need to cleanup disk space.  .venv/ is a standard folder naming convention for virtual environments and should already be included on any standard Python .gitignore file.
```

