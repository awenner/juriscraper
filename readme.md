What is this?
=============
Juriscraper is a scraper library started several years ago that is gathers 
judicial opinions and oral arguments in the American court system. It is 
currently able to scrape:

  - opinions from all major appellate Federal courts
  - opinions from all state courts of last resort (typically their "Supreme 
    Court")
  - oral arguments from all appellate federal courts that offer them

Juriscraper is part of a two-part system. The second part is your code, which
calls Juriscraper. Your code is responsible for calling a scraper, downloading 
and saving its results. A reference implementation of the caller has been 
developed and is in use at [CourtListener.com][2]. The code for that caller 
can be [found here][1]. There is also a basic sample caller [included in 
Juriscraper][5] that can be used for testing or as a starting point when 
developing your own.

Some of the design goals for this project are:

 - extensibility to support video, oral argument audio, etc.
 - extensibility to support geographies (US, Cuba, Mexico, California)
 - Mime type identification through magic numbers
 - Generalized architecture with minimal code repetition
 - XPath-based scraping powered by lxml's html parser
 - return all meta data available on court websites (caller can pick what it needs)
 - no need for a database
 - clear log levels (DEBUG, INFO, WARN, CRITICAL)
 - friendly as possible to court websites


Installation & dependencies
===========================
    # install the dependencies
    sudo apt-get install python-pip  # If you don't have it already...
    sudo apt-get install libxml2-dev libxslt-dev  # In Ubuntu prior to 14.04 this is named libxslt-devel
    sudo pip install chardet==1.0.1
    sudo pip install requests==2.2.1
    sudo pip install lxml==3.0.1
    sudo pip install python-dateutil==1.5  # Newer have known incompatibilities with Python < 3
    
    # Optionally, you may install cchardet for faster tests and better 
    # performance. Note that unless you have an extremely fast CPU, this is 
    # a requirement for tests to pass as they have speed requirements.
    sudo pip install cchardet==0.3.5
    
    # Create a directory for logs
    sudo mkdir /var/log/juriscraper/

    # Install selenium and PhantomJS
    sudo pip install selenium
    wget https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-1.9.7-linux-x86_64.tar.bz2
    tar -x -f phantomjs-1.9.7-linux-x86_64.tar.bz2
    sudo mkdir -p /usr/local/phantomjs
    sudo mv phantomjs-1.9.7-linux-x86_64/bin/phantomjs /usr/local/phantomjs
    rm -r phantomjs-1.9.7*  # Cleanup
    
    # Finally, install the code
    sudo mkdir /usr/local/juriscraper
    cd /usr/local/juriscraper
    git clone https://github.com/freelawproject/juriscraper.git .

    # add Juriscraper to your python path (in Ubuntu/Debian)
    sudo ln -s /usr/local/juriscraper /usr/lib/python2.7/dist-packages/juriscraper

    # create a directory for logs
    sudo mkdir -p /var/log/juriscraper


Joining the project as a developer
==================================
We use a few tools pretty frequently while building these scrapers. The first is
[a sister project called xpath-tester][3] that helps debug XPath queries.
xpath-tester can be installed locally in a few minutes or is available at
[http://xpath.courtlistener.com][7].

We also generally use Eclipse with the PyDev and Aptana tools installed or 
Intellij with PyCharm installed. These are useful because they allow syntax 
highlighting, code inspection, and PyLint integration. Intellij is particularly
strong in this area and a license is available to interested contributors.

For scrapers to be merged:

 - `python tests/tests.py` must pass, listing the results for any new scrapers
 - a *_example* file must be included (this is needed for the tests to
   run your code -- see examples of these files next to any current scraper).
 - your code should be [PEP8][4] compliant with no major Pylint problems or
Intellij inspection issues.
 - your code should efficiently parse a page, returning no exceptions or
   speed warnings

When you're ready to develop a scraper, get in touch, and we'll find you one
that makes sense and that nobody else is working on. Alternatively, we have
[a list][6] of courts that you can browse yourself. There are templates for new
scrapers [here][10] and [here][11].

When you're done with your scraper, fork this repository, push your changes 
into your fork, and then send a pull request for your changes. Be sure to
remember to update the `__init__.py` file as well, since it contains a list of
completed scrapers.

Before we can accept any changes from any contributor, we need a signed and
completed Contributor License Agreement. You can find this agreement in the
root of the repository. While an annoying bit of paperwork, this license is for
your protection as a Contributor as well as the protection of Free Law Project
and our users; it does not change your rights to use your own Contributions for
any other purpose.


Usage
======
The scrapers are written in Python, and can can scrape a court as follows:

    from juriscraper.opinions.united_states.federal_appellate import ca1

    # Create a site object
    site = ca1.Site()

    # Populate it with data, downloading the page if necessary
    site.parse()

    # Print out the object
    print str(site)
    
That will print out all the current meta data for a site, including links to 
the objects you wish to download (typically opinions). If you download those
opinions, we also recommend running the `_cleanup_content()` method against the
items that you download (PDFs, HTML, etc.). See the `sample_caller.py` for an
example.

It's also possible to iterate over all courts in a Python package, even if
they're not known before starting the scraper. For example:

    court_id = 'juriscraper.opinions.united_states.federal'
    scrapers = __import__(court_id,
                          globals(),
                          locals(),
                          ['*']).__all__
    for scraper in scrapers:
        mod = __import__('%s.%s' % (court_id, scraper),
                         globals(),
                         locals(),
                         [scraper])
        site = mod.Site()

This can be useful if you wish to create a command line scraper that iterates
over all courts of a certain jurisdiction that is provided by a script or a user.
See `lib/importer.py` for an example that's used in the sample caller.

Development of a `to_xml()` or `to_json()` method has not yet been completed, as
all callers have thus far been able to work directly with the Python objects. We
welcome contributions in this area.


Tests
=====
We got that! You can (and should) run the tests with `python tests/tests.py`. 
This will iterate over all of the *_example_* files and run the scrapers 
against them.


Version History
===============
**Past**

 - 0.1 - Supports opinions from all 13 Federal Circuit courts and the U.S. Supreme Court
 - 0.2 - Supports opinions from all federal courts of special jurisdiction (Veterans, Tax, etc.)
 - 0.8 - Supports oral arguments for all possible Federal Circuit courts.
 - 0.9 - Supports all state courts of last resort (typically the "Supreme" court)

**Current**

 - 1.0 - Support opinions from for all possible federal bankruptcy appellate panels (9th and 10th Cir.) 

**Future Roadmap**

 - 1.5 - Support opinions from for all intermediate appellate state courts
 - 1.6 - Support opinions from for all courts of U.S. territories (Guam, American Samoa, etc.)
 - 2.0 - Support opinions from for all federal district courts with non-PACER opinion listings
 - 2.5 - Support opinions from for all federal district courts with PACER written opinion reports (+JPML)
 - 2.6 - Support opinions from for all federal district bankruptcy courts
 - 3.0 - For every court above where a backscraper is possible, it is implemented.

**Beyond**
 - Support video, additional oral argument audio, and transcripts everywhere available
 - Add other countries, starting with courts issuing opinions in English.
 

License
========
Juriscraper is licensed under the permissive BSD license.

[1]: https://github.com/freelawproject/courtlistener/blob/master/alert/scrapers/management/commands/cl_scrape_and_extract.py
[2]: http://courtlistener.com
[3]: https://github.com/mlissner/lxml-xpath-tester
[4]: http://www.python.org/dev/peps/pep-0008/
[5]: https://github.com/freelawproject/juriscraper/blob/master/sample_caller.py
[6]: https://github.com/freelawproject/juriscraper/wiki/Court-Websites
[7]: http://xpath.courtlistener.com
[8]: http://phantomjs.org
[9]: http://phantomjs.org/download.html
[10]: https://github.com/freelawproject/juriscraper/blob/master/opinions/opinion_template.py
[11]: https://github.com/freelawproject/juriscraper/blob/master/oral_args/oral_argument_template.py
