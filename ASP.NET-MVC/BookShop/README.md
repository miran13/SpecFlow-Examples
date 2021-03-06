SpecFlow BookShop Sample
========================

This solution contains an end-to-end sample to show how to use SpecFlow for 
ASP.NET MVC applications.

You can find more information about SpecFlow at [http://www.specflow.org/](http://www.specflow.org).

Prerequisites to run the application
====================================

- [Visual Studio 2017](https://www.visualstudio.com/downloads/)
- Microsoft SQL Server localdb version 11.0
- [SpecFlow](http://www.specflow.org/) 2.2.1 or higher

Setup Application
=================

- Create a database and initialize the table structure by executing `create_db.sql`
  (This script will create and setup a database called *BookShop*)
- Update the connection string in
  `web.config` (project `BookShop.Mvc`) and `app.config` (project `BookShop.AcceptanceTests`)
  in the `<add name="BookShopEntities"/>` element:
  The default setting points to a (localdb) instance:
  `connectionString="metadata=res://*/Models.BookShop.csdl|res://*/Models.BookShop.ssdl|res://*/Models.BookShop.msl;provider=System.Data.SqlClient;provider connection string=&quot;Data Source=(localdb)\v11.0;Initial Catalog=BookShop;Integrated Security=True;MultipleActiveResultSets=True&quot;"`
  You might need to change the data source property, if you're not runnning localdb,
  which can be easily done with a solution wide search and replace, eg.:
    `Data Source=(localdb)\v11.0; -> Data Source=.\SQLEXPRESS;`
- Set the `BookShop.Mvc` project as startup project and run the application. You
  should see some books on the start page of the app.

Automated SpecFlow Acceptance Tests
===================================

With SpecFlow you can define the acceptance criteria in *.feature* files, that 
can be automatically validated through acceptance tests. 
The `BookShop.AcceptanceTests` project contains the feature files for this application,
which describe the implemented behaviour as acceptance criteria (SpecFlow scenarios with examples).
Step definitions (folder *StepDefinitions*) define, how individual scenario steps should be automated.
SpecFlow generated executable unit tests from the defined acceptance criteria scenarios
in the feature files and the step definitions, that you need to define and implement.
The generated unit tests are shown as sub-items of each feature file (e.g. `US01_BookSearch.feature.cs`).

When you build the solution and open the Visual Studio Test Explorer, you should
see all the tests generated from the feature files.

In Visual Studio 2010 you'll see the individual feature file scenario titles
as test method name (Pascal case with blanks in the title removed).
In Visual Studio 2012 (which provides better test runner integration), you'll directly
see the scenario title names as written in the feature files.

Unit tests are always generated by SpecFlow from the feature files -
when working with the solution from the IDE the generation is triggered when you save a feature file,
when building the solution on a build server, SpecFlow automatically updates the unit tests as an msbuild task.
You should never manually modify the generated unit tests. They are just an artifact generated by the build!

Executing SpecFlow Acceptance Tests
===================================

In the Visual Studio Test Explorer select *Run All* tests.

If you've configured the database connection correctly, all tests should pass (green).

A test execution report is generated automatically.
Open the Visual Studio "Output" window and select "Show output from: Tests".
A hyperlink to an HTML execution report should be shown there, which gives an overall test result as well as a break down of each individual scenario execution.

As SpecFlow is not a unit test runner on its own, it can generate unit tests for a number of third party unit test runners like MsTest, NUnit, XUnit and SpecRun.
(check the SpecFlow documentation for an up-to-date list of supported unit test runners)

The sample project is configured to generate unit tests for SpecRun, which is a
unit test runner provided by TechTalk which is specialized for running acceptance/integration tests
(like written in SpecFlow but also without SpecFlow).
SpecRun is provided as a free evaluation version with this example. The only
limitation of the evaluation is, that your tests will be delayed by an arbitrary
time span before being executed.
To find out more about SpecRun and its further benefits for automated acceptance testing
please visit [http://www.specrun.com](http://www.specrun.com).

You can easily switch the sample project to use MsTest instead of SpecRun:
Make the following modifications in "App.config" of the `BookShop.AcceptanceTests` project:

- Comment (or remove) the following line:
`<unitTestProvider name="SpecRun" />`
- Uncomment (or add) the following line instead:
`<unitTestProvider name="MsTest" />`
- You can also comment (or remove) the SpecRun plugin from the `<plugins/>` element

After saving these changes to app.config, SpecFlow will re-generate the unit tests
of all feature files for MsTest instead of SpecRun. When you re-run all tests from
Test Explorer, the tests will be executed by MsTest.

Similarly, you can switch to other unit test providers (such as [NUnit](http://nunit.org/), [XUnit](https://xunit.github.io/), etc.).
Please refer to the SpecFlow documentation to find out more about supported configuration options.

If you do not use SpecRun, you can also remove the SpecRun NuGet package from the `BookShop.AcceptanceTests` project.

Using SpecFlow for UI automation
================================

SpecFlow is completely independent of what automation interface is used.

This example automates the tests directly through the Controller of the
MVC web application (this is also sometimes called *automation below the skin*).

Automating below the skin provides several benefits:
less brittle tests, less efforts for automation, better performance of the test suite

However, sometimes behaviour that should be validated cannot be observed
on the controller level, but just on the UI. In that case, some kind of
UI automation has to be implemented.

To demonstrate this, the sample implements an alternative automation for all the
`US01_BookSearch.feature` scenarios using Selenium.

To enable the Selenium UI automation, you need to add (uncomment) the
`@web` tag on the `US01_Booksearch` feature file.

You also need to have the correct version of FireFox installed, that can be
driven by the Selenium version used in this example. It might be necessary
to update FireFox or the Selenium version used in this sample, to make
the UI automation work.

As the UI automation relies on a running instance of the application,
you need to first run the web application, before you run the SpecFlow tests
(e.g. start the web project using [Ctrl-F5] and validate the web application is working).

After that, if you have enabled the `@web` tag, and run all the SpecFlow scenarios,
the scenarios of `US01_BookSearch.feature` will be automated through the UI
with Selenium, while the other scenarios are still automated through the controller.

Notice that the phrasing of the scenarios didn't have to be changed, in order
to automate on a different layer. This is a good practices, as SpecFlow scenarios
shouldn't express technical details of the automation, but the intention and
behaviour to be validated.

The different automation is implemented in a separate step-definition assembly,
which is located in the `BookShop.WebTests.Selenium` project.
This project provides alternative step-definitions that use Selenium for automation,
and which are scoped to scenarios having the `@web` tag (`[Binding, Scope(Tag = "web")]`).

Additional step definition assemblies can be registered in the app.config of the
SpecFlow project (`BookShop.AcceptanceTests`) using the `<stepAssemblies/>` element, e.g.:
`<stepAssembly assembly="BookShop.WebTests.Selenium" />`
