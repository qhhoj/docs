# PDF generation of problem statements

The QHHOJ supports rendering problem statements to PDF. This can be useful in the case of on-site contests, where contestants receive paper versions of the problems.

For example, [here](https://qhhoj.com/problem/mochi/pdf) is a generated PDF of
[this problem](https://qhhoj.com/problem/mochi).

PDF generation is backed by a related project, [Pdfoid](https://github.com/qhhoj/pdfoid). Pdfoid interfaces with [Selenium](https://www.selenium.dev/) to provide a REST endpoint for PDF rendering.

## Installing Pdfoid

First, clone the Pdfoid repository, and install it along with its dependencies into a new virtualenv.

```shell-session
$ git clone https://github.com/qhhoj/pdfoid.git
$ cd pdfoid
$ python3 -m venv env
$ . env/bin/activate
$ pip install -e .
```

Install `exiftool`, which is used to set PDF titles.

```shell-session
$ apt install exiftool
```

Since Pdfoid only works with Chrome 121 or older, we need to download and install an older version of Google Chrome. 

```shell-session
$ curl -O https://qhhoj.com/materials/docs/google-chrome-stable_120.0.6099.71-1_amd64.deb
$ sudo dpkg -i google-chrome-stable_120.0.6099.71-1_amd64.deb 
$ sudo apt --fix-broken install  # fix dependencies problems
```

Install [ChromeDriver](https://chromedriver.chromium.org/downloads), a special version of the Chromium engine needed by Selenium to create PDFs. We'll use a special Python package called [chromedriver-autoinstaller](https://pypi.org/project/chromedriver-autoinstaller/) to install ChromeDriver.

```shell-session
$ python3
Python 3.12.4 (main, Jun  6 2024, 18:26:44) [GCC 11.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from selenium import webdriver
>>> import chromedriver_autoinstaller
>>> chromedriver_autoinstaller.install()
'/path/to/pdfoid/env/lib/python3.10/site-packages/chromedriver_autoinstaller/120/chromedriver'
```

Finally, type `exit()` or press Ctrl + D to exit the `python3` command. Please make sure to take a note of the output path of the `chromedriver` binary for later.  

## Running Pdfoid

To start the Pdfoid server, run:

```shell-session
$ export CHROME_PATH=$(which google-chrome)
$ export EXIFTOOL_PATH=$(which exiftool)
$ export CHROMEDRIVER_PATH=<path to the chromedriver binary from the previous step>
$ env/bin/pdfoid --port=<port>
```

The environment variables `CHROME_PATH` and `EXIFTOOL_PATH` are not necessary if the `google-chrome` and `exiftool` executables are present in `$PATH`, as they should be if you followed the installation instructions above. However, you must set the environment variable `CHROMEDRIVER_PATH` for Pdfoid to work.

To test, start Pdfoid with `--port=8887`. Then, we can request a render of a simple HTML document.

```html
<div>Welcome to QHHOJ!</div>
```

The response should contain JSON, with a Base64-encoded PDF inside.

```shell-session
$ curl -d "title=QHHOJ&html=Welcome to QHHOJ!" -X POST -H "Content-Type: application/x-www-form-urlencoded" http://localhost:8887
{
    "success": true,
    "pdf": "JVBERi0xLjQKJdPr6eEKMSAwIG9iago8PC9DcmVhdG9yIChDaHJvbWl1bSkK..."
}
```

?>  PDFs statements rendered by QHHOJ uses the Lato font by default. Therefore, installing the Lato font on the machine running Pdfoid will provide optimal rendering quality &mdash; otherwise, a fallback font will be used and statements will look subpar.

## Configuration

Configuring QHHOJ to generate PDFs with Pdfoid can be done by adding the following lines to your `local_settings.py`.

```python
# The URL Pdfoid is running on.
DMOJ_PDF_PDFOID_URL = 'http://localhost:8887'

# Optional, cache location for generated PDFs. You should consider using
# something more persistent than /tmp, since PDF generation is an expensive
# operation. If omitted, no cache will be used.
DMOJ_PDF_PROBLEM_CACHE = '/tmp'

# Optional, URL serving DMOJ_PDF_PROBLEM_CACHE with X-Accel-Redirect. This is
# recommended to have nginx serve PDFs, rather than uWSGI. To enable this,
# uncomment the line below, as well as the corresponding section in the sample
# nginx configuration file.
#DMOJ_PDF_PROBLEM_INTERNAL = '/pdfcache'
```

Restart QHHOJ for changes to take effect.

## Troubleshooting

### "View as PDF" button doesn't show up

If a "View as PDF" button does not show up on the problem page, make sure that the `DMOJ_PDF_PDFOID_URL` variable is set.

### "View as PDF" button shows up

If a "View as PDF" button shows up, but generation fails, an error log should be displayed in the browser. This log will also be captured by the `judge.problem.pdf` Django log handler. Depending on the error, explicitly setting the Pdfoid environment variables `CHROME_PATH` to the path of the Chromium binary and `CHROMEDRIVER_PATH` to the path of the ChromeDriver binary may alleviate the problem.

For other errors, take a look at the [Selenium documentation](https://www.selenium.dev/documentation/webdriver/), specifically the [common exceptions](https://www.selenium.dev/selenium/docs/api/py/common/selenium.common.exceptions.html) section.
