# morph-browser

Lightweight web browser tailored for Ubuntu,
based on the Qt WebEngine  and using the Ubuntu UI components.
It requires Qt 5.9 to build and run.


= Building =

The build system uses cmake.
To compile, simply invoke cmake and then make:

    $ cmake .
    $ make

The application can also be cross compiled for an ARM target on a X86 host.
To do that, just pass this additional parameter to cmake:

    $ cmake -DCMAKE_TOOLCHAIN_FILE=cmake/ubuntu-arm-linux-gnueabihf.cmake .

== Building for ubuntu touch ==

Simply use [clickable](http://clickable.bhdouglass.com/en/latest/) command line

	$ clickable

= Running =

webbrowser-app can be run from the development branch without the need to
install any files. Just run:

    $ ./src/app/webbrowser/webbrowser-app

The executable accepts command line switches and parameters. To find out which,
just run:

    $ ./src/app/webbrowser/webbrowser-app --help


= Build with docker =

First we need to build the image

   $ docker build -t ubports_xenial .

Then we can build the app

   $ docker run --privileged -ti --rm -e DISPLAY=:0 -v /var/run/dbus:/var/run/dbus -v /tmp/.X11-unix:/tmp/.X11-unix -v `pwd`:/home/developer/ubports_build ubports_xenial bash -c "cmake . && make"

Now we can run the app
   
   $ docker run --privileged -ti --rm -e DISPLAY=:0 -v /var/run/dbus:/var/run/dbus -v /tmp/.X11-unix:/tmp/.X11-unix -v `pwd`:/home/developer/ubports_build ubports_xenial bash -c "./src/app/webbrowser/webbrowser-app" 

= Unit tests =

To run the unit tests, you can use the commands below:

    $ make test

      - or -

    $ ctest


= Automated UI tests =

webbrowser-app uses autopilot (https://launchpad.net/autopilot) to test its UI.
To run the tests locally, you will need to install python3-autopilot and
autopilot-qt5.
Then do the following:

    $ cd tests/autopilot/
    $ autopilot3 run webbrowser_app

You can get a list of all available tests with the following command:

    $ autopilot3 list webbrowser_app

In order to run the tests in a virtual machine with an environment closer to
what a user will get in Ubuntu Touch, see the Dep8 tests section.


= Code coverage =

To generate a report with detailed code coverage, you need to re-run cmake with
"CMAKE_BUILD_TYPE=coverage":

    $ cmake -DCMAKE_BUILD_TYPE=coverage .
    $ make
    $ make test
    $ make coverage

This will generate a coverage report in XML format (coverage.xml) and an
interactive human-readable report in HTML format (coveragereport/index.html).


= Dep8 tests =

Dep8 tests exercise the package "as-installed".

Currently, the webbrowser-app has one suite of dep8 tests that uses autopilot
(https://launchpad.net/autopilot) to test from the point of view of the user.

To run the tests you will need autopkgtest:

    $ sudo apt-get install autopkgtest

You can use multiple test beds to execute the tests. Below you will find
instructions to run them in a virtual machine
You can find more information with:

    $ man adt-run

== Run dep8 tests ==

To run the tests in a qemu virtual machine, you will first have to create it
(see /usr/share/doc/autopkgtest/README.running-tests.rst.gz). We output the
image to ~/ rather than the current directory, so it will be in a safer
place to avoid rebuilding images every time. You can store it in any
directory you wish. This image is better consumed "fresh", building it daily
will avoid long updates/upgrades when running the tests.

    $ adt-buildvm-ubuntu-cloud -r $(lsb_release -c -s) -a amd64 -o ~/

Then run the tests using adt-run with the qemu virtualization host against
the current archive.

    $ adt-run -B -U --unbuilt-tree . \
      -o ~/adt-browser-test/$(date +%Y-%m-%d-%H-%M) \
      --- qemu ~/adt-$(lsb_release -c -s)-amd64-cloud.img

The tests can also be run on a local phone.

    $ adt-run -B -U --unbuilt-tree . \
      -o ~/adt-browser-test/$(date +%Y-%m-%d-%H-%M) \
      --- ssh -s adb -- -p <password> --serial <ADB_SERIAL> 

== Examine the dep8 autopilot results ==

To examine the test results, which are in subunit format, additional tools are
required, such as trv (https://launchpad.net/trv).

You can find the results file in the directory
~/adt-browser-test/$(date +%Y-%m-%d-%H-%M)/artifacts.


= Settings =

webbrowser-app supports a limited set of custom settings, persisted on disk in
the following INI-like file:

    $HOME/.config/webbrowser-app/webbrowser-app.conf

The following keys are supported:

 - 'homepage': a URL that the browser will open when launched if no URL is
   specified on the command line

 - 'searchengine': a custom search engine specification, looked up in
   $HOME/.local/share/webbrowser-app/searchengines/{value}.xml and following
   the OpenSearch document description format
   (http://www.opensearch.org/Specifications/OpenSearch/1.1)

 - restoreSession: whether to restore the previous browsing session at startup
   (defaults to true)

