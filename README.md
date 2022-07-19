# CAMPD Tools Proxy

[![License](https://img.shields.io/github/license/US-EPA-CAMD/campd-tools-proxy)](https://github.com/US-EPA-CAMD/campd-tools-proxy/blob/develop/LICENSE)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=US-EPA-CAMD_campd-tools-proxy&metric=alert_status)](https://sonarcloud.io/dashboard?id=US-EPA-CAMD_campd-tools-proxy)
[![Develop CI/CD](https://github.com/US-EPA-CAMD/campd-tools-proxy/workflows/Develop%20Branch%20Workflow/badge.svg)](https://github.com/US-EPA-CAMD/campd-tools-proxy/actions)
[![Release CI/CD](https://github.com/US-EPA-CAMD/campd-tools-proxy/workflows/Release%20Branch%20Workflow/badge.svg)](https://github.com/US-EPA-CAMD/campd-tools-proxy/actions)
![Issues](https://img.shields.io/github/issues/US-EPA-CAMD/campd-tools-proxy)
![Forks](https://img.shields.io/github/forks/US-EPA-CAMD/campd-tools-proxy)
![Stars](https://img.shields.io/github/stars/US-EPA-CAMD/campd-tools-proxy)
[![Open in Visual Studio Code](https://open.vscode.dev/badges/open-in-vscode.svg)](https://open.vscode.dev/US-EPA-CAMD/campd-tools-proxy)

## Description
CAMPD tools proxy for correcting the behavior of R Shiny apps that are sensitive to the URLs used to make requests, and misbehave when Cloud Foundry serves them from a route with a `--path`, either not responding or trying to find their assets in an unrelated location such as the root "/" path.

### That's too vague... What's an example of an app that misbehaves?
Running [R Shiny](https://shiny.rstudio.com/) apps on [Cloud Foundry](https://www.cloudfoundry.org/) works great thanks to the [R Buildpack](https://docs.cloudfoundry.org/buildpacks/r/index.html)... until you try to map your R Shiny app a route to with a `--path`! Then you'll see a `404 Not Found` message from the Shiny app. 

If you provide a `uiPattern` parameter matching the path when you set up the app object in your R code, you'll see the app respond, but request JS and CSS assets from a `/shared` path. Those assets won't be found, and the app will appear broken.

This means R Shiny apps break two of the [12-factors](https://12factor.net/config) for well-behaved applications:

* [Configuration should be separate from the app](https://12factor.net/config): A Shiny app should not care about how requests are routed to it, or whether there's a path in the URL.
* [Dependencies should be isolated](https://12factor.net/dependencies): A Shiny app should not request JS and CSS from an unrelated app or location.

## What's the solution?

We can restore these two factors by isolating the misbehaving application, then using a proxy app as a front-end to strip the path from the request before it reaches the problem app. The misbehaving app will see all requests arriving via `/` instead of a subpath, avoiding the broken behavior. 

Here we use the [`nginx-buildpack`](https://docs.cloudfoundry.org/buildpacks/nginx/index.html) to implement such a proxy, referencing [an example from the Shiny documentation](https://support.rstudio.com/hc/en-us/articles/213733868-Running-Shiny-Server-with-a-Proxy).

## Using the proxy
1. Map the misbehaving app to a route on an `.internal` domain, eg `<appname>.apps.internal`.
1. Clone this repo and copy `manifest.yml-dist` to `manifest.yml`.
1. Edit `manifest.yml` to set the application name and hostname+domain path.
1. Run `cf push`.
1. Add a network policy enabling the proxy to reach the misbehaving app. For example:
  `cf add-network-policy subpath-proxy --destination-app <appname>`
1. Open `example.appdomain/<appname>` in your browser.

## License & Contributing
This project is licensed under the MIT License. We encourage you to read this project’s [License](LICENSE), [Contributing Guidelines](CONTRIBUTING.md), and [Code of Conduct](CODE-OF-CONDUCT.md).

## Disclaimer
The United States Environmental Protection Agency (EPA) GitHub project code is provided on an "as is" basis and the user assumes responsibility for its use. EPA has relinquished control of the information and no longer has responsibility to protect the integrity , confidentiality, or availability of the information. Any reference to specific commercial products, processes, or services by service mark, trademark, manufacturer, or otherwise, does not constitute or imply their endorsement, recommendation or favoring by EPA. The EPA seal and logo shall not be used in any manner to imply endorsement of any commercial product or activity by EPA or the United States Government.