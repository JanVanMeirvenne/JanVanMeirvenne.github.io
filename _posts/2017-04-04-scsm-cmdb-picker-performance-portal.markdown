---
layout:     post
title:      "Slow cmdb picker controls in request forms"
subtitle:   "Troubleshooting SCSM portal performance"
date:       2017-04-04 12:00:00
author:     "Jan Van Meirvenne"
header-img: "img/scsmbg.jpg"
comments: true
---

# Slow loading of CMDB picker controls in the SCSM portal

## The issue

When a service offering contains a CMDB input with a high amount of objects behind it, it may cause issues in the Self-Service portal.

A request offering form initially does not load the objects in a CMDB control until the 'Refresh' button is clicked.

When the button is clicked, the portal will perform the following actions:

* Perform a background query to the backend server and generate a basic html table containing all objects specified by the CMBD picker filter
* Inject the html table in the current form page
* Use client-side javascript to transform the basic table into a rich one that allows for sortinng, pagination, search,...

These actions can take some time, from a few seconds to entire minutes depending on the configuration of the platform. The acceptable delay for this is subjective of course, but a minute of waiting is seldom pleasant.

In our situation, we had a wait time of 50 seconds between clicking the 'refresh'-button and having the CMDB items visible. This was by using IE11 (company required browser). With Chrome, this was 20 seconds.

## The cause and fix

The cause of the slowdown can be 2-fold:

* Server-Based (occurs while data is gathered)
* Client-Based (occurs while data is processed by the browser for visualization)

You can determine the slowdown type by doing some profiling in the browser, but there is a simpler way:

* Click the 'refresh' button, a waiting animation will be shown
* While the animation is running, the form is waiting for the server to provide data
* When the animation stops running, and until the table is shown, the browser is busy processing data

So it is important to identify your slowdown problem as a mainly server- or client-side issue.

### Server-Side

Server-side performance issues can usualy be attributed to 2 causes:

#### Slow SCSM / DB servers

The portal connects to the SCSM management server to request the CMDB data. If either the management server, or the operational database behind it are having performance issues, you will always notice slowdowns in the portal.

The only way to resolve this is to make sure that all involved systems are adequately sized. In addition, having a healthy SCSM and SQL hygiene makes sure that performance is optimal.

#### Complex CMDB picker query

When configuring your request offering, make sure to choose a type projection that contains as few components as possible. This reduces the amount of time needed to query the data from the database. Choosing the 'Computer (Advanced)' projection for example is not a good idea as it contains the computer-object and also all relations it has to users / tickets / .... Which is bot han overkill for your requirements, and your query performance.

Also, you should use filters to remove unneeded items (for example, do not show devices that are not yet deployed)

### Client-Side

Client-Side performance is controlled by 2 factors:

* The browser, more specifically, its scripting engine
* The type and version of the scripts used to power the web page UI features

#### Browser

The portal makes heavy use of JQuery plugins (a framework around plain JavaScript, which is a coding language used to interact with HTML pages). All browsers come with a scripting engine that during the lifetime of a web page, interpret scripts to manipulate the page. One of these manipulations for example is converting received SCSM data into a nice looking table in the request form.

This scripting engine can be different for each browser. This results in differencing performance depending on the engine's efficience and age.

Why are Chrome, Edge and Firefox faster than IE11? Because their scripting engines are much more optimized for modern websites, while IE11's engine is more targetted towards backwards compatibility.

This is why the SCSM portal is usually more snappy to use in Chrome or Edge for example.

Whether you can change the browser in your company is entirely up to its policy. If this is not the case, read on for some 'tweaks' you can do to get the CMDB picker more IE-friendly.

#### Scripts

> Note: the following changes are probably unsupported by Microsoft, and should be used at your own-risk

The SCSM portal uses JQuery plugins to provide a rich user experience. The used JQuery versions are dated from 2015 (2.1.4). The most recent version is 3.2.0, with the chance of containing some performance improvements.

To update the JQuery plugins the following should be done:

Download the [3.2.0 minified version](https://code.jquery.com/jquery-3.2.0.min.js) and replaced both the 'jquery.min.js' and the 'jquery.js' files in the 'Content\\JS' folder with this file (which you should rename accordingly). Back up the original files just in case. The minified version has all code compressed to a single line, to improve parsing speed when it is interpreted by the browser.

Download the [1.10.13 minified version of the datatables plugin](https://cdn.datatables.net/1.10.13/js/jquery.dataTables.min.js) and overwrite the existing 'jquery.datatTables.min.js' file in 'Content\\JS'. Unless you forget: backup!


This provided a slight, but noticable improvement in loading performance, but we are not done yet.

The JQuery data tables plugin is used to convert the server data to nice looking tables. This conversion is a heavy process as the order/search/layout functions of the table are calculated for each object in the returned data. Combine this with an IE11 scripting engine, and you found yourself a bottleneck.

Luckily, the plugin can be tweaked to make it less resource-intensive:

* Open the file 'Views\\Home\\MakeForm.cshtml'
* Look for the following code:

```javascript
// If refresh query button is clicked
            $('.queryPickButton').on('click', function (event) {
                event.stopPropagation();
                var table;
                var clickedElement = $(this).attr("data-idSource");
                var docHeight = $(document).height();

                var overlayDiv = $("<div id='overlay'></div>").height(docHeight);
                $("body").append(overlayDiv);
                $('#page_loader').show();

                $.ajax({
                    url: "/Home/Query",
                    type: 'POST',
                    data: $('form#formLink').serialize() + "&clickedElement=" + clickedElement + "&ROGuid=" + '@requestItem["BMEId"]',
                    success: function (data) {
                        $('#page_loader').hide();
                        $("body #overlay").remove();

                        $('div#' + clickedElement).empty();
                        $('div#' + clickedElement).html(data);

                        // Clear the old values from hidden input
                        $('input:hidden[name="' + clickedElement + '"]').attr("value", "");

                        table = $('div#' + clickedElement).find('table').DataTable({
                            "paging": true,
                            "searching": true,
                            "ordering": true,
                            "scrollY": "35em",
                            "scrollCollapse": true,
                            "autoWidth": true,
                            "language": {
                                "emptyTable": "@Resources.SelfServicePortalResources.NoSearchRecords",
                                "zeroRecords": "@Resources.SelfServicePortalResources.NoSearchRecords",
                                "search": "@Resources.SelfServicePortalResources.Search:",
                                "info": "@Resources.SelfServicePortalResources.Showing _START_ to _END_ of _TOTAL_ entries",
                                "infoEmpty": "@Resources.SelfServicePortalResources.Showing 0 to 0 of 0 @Resources.SelfServicePortalResources.Enteries",
                                "infoFiltered": "(@Resources.SelfServicePortalResources.Filtered from _MAX_ @Resources.SelfServicePortalResources.Enteries)",
                                "lengthMenu": "@Resources.SelfServicePortalResources.Show _MENU_ @Resources.SelfServicePortalResources.Enteries",
                                "paginate": {
                                    "first": "@Resources.SelfServicePortalResources.First",
                                    "last": "@Resources.SelfServicePortalResources.Last",
                                    "next": "@Resources.SelfServicePortalResources.Next",
                                    "previous": "@Resources.SelfServicePortalResources.Previous"
                                },
                            }
                        }); // end of datatable

                        // Add the handler to associative array
                        // Had to be within success block
                        queryTableHandler[clickedElement] = table;

                    } // end of success
                }); // end of ajax
```

* Modified it like this (explanation in comments):

```javascript
// If refresh query button is clicked
            $('.queryPickButton').on('click', function (event) {
                event.stopPropagation();
                var table;
                var clickedElement = $(this).attr("data-idSource");
                var docHeight = $(document).height();

                var overlayDiv = $("<div id='overlay'></div>").height(docHeight);
                $("body").append(overlayDiv);
                $('#page_loader').show();

                $.ajax({
                    url: "/Home/Query",
                    type: 'POST',
                    data: $('form#formLink').serialize() + "&clickedElement=" + clickedElement + "&ROGuid=" + '@requestItem["BMEId"]',
                    success: function (data) {
                        $('#page_loader').hide();
                        $("body #overlay").remove();

                        $('div#' + clickedElement).empty();
                        $('div#' + clickedElement).html(data);

                        // Clear the old values from hidden input
                        $('input:hidden[name="' + clickedElement + '"]').attr("value", "");

                        table = $('div#' + clickedElement).find('table').DataTable({
                            "paging": true,
							"deferRender": true, // improves rendering
							"orderClasses": false, // disable some layout calculations that make sorting visually appealing.
                            "searching": true,
                            "ordering": false, // if your end-users solely use the search function to pinpoint the CMDB object, the sort-function is not needed
                            "scrollY": "35em",
                            "scrollCollapse": true,
                            "autoWidth": false, // disable auto-calculation of the column width
							"columnDefs" : [ // define a column width of 20%. Might need tweaking if there are visual artifacts.
								{ "width":"20%" }
							],
                            "language": {
                                "emptyTable": "@Resources.SelfServicePortalResources.NoSearchRecords",
                                "zeroRecords": "@Resources.SelfServicePortalResources.NoSearchRecords",
                                "search": "@Resources.SelfServicePortalResources.Search:",
                                "info": "@Resources.SelfServicePortalResources.Showing _START_ to _END_ of _TOTAL_ entries",
                                "infoEmpty": "@Resources.SelfServicePortalResources.Showing 0 to 0 of 0 @Resources.SelfServicePortalResources.Enteries",
                                "infoFiltered": "(@Resources.SelfServicePortalResources.Filtered from _MAX_ @Resources.SelfServicePortalResources.Enteries)",
                                "lengthMenu": "@Resources.SelfServicePortalResources.Show _MENU_ @Resources.SelfServicePortalResources.Enteries",
                                "paginate": {
                                    "first": "@Resources.SelfServicePortalResources.First",
                                    "last": "@Resources.SelfServicePortalResources.Last",
                                    "next": "@Resources.SelfServicePortalResources.Next",
                                    "previous": "@Resources.SelfServicePortalResources.Previous"
                                },
                            }
                        }); // end of datatable

                        // Add the handler to associative array
                        // Had to be within success block
                        queryTableHandler[clickedElement] = table;

                    } // end of success
                }); // end of ajax
```

* Backup the original file before doing this! Also, backup your changes before installing a portal update, as the file will probably be overwritten.

This reduced the load time of ~50 seconds in IE11 to ~17! This was done in a dev environment where the server query took 10 seconds to get the data, so in a production environment with extra sizing, the milage may even be better!


## References

* [JQuery](http://jquery.com/)
* [JQuery Data Tables](https://www.datatables.net/)
* [SCSM Hardware Requirements](https://technet.microsoft.com/system-center-docs/system-requirements/system-requirements)
* [System Center SQL Best Practices, old but gold](http://www.hasmug.com/wp-content/uploads/2012/10/07-201210-Oct-SQL-Server-Optimization-and-Best-Practices-for-System-Center-Administrators-Kevin-Holman.pdf)