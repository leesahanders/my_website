# Computer System Validation (CSV)

## Introduction

Another important thing to understand is Computer System Validation (CSV). 

CSV is used in order to ensure that a computerized system is meeting the expectation and produces correct results for the intended purpose/use case. 

In Pharma this is extremely important for any aspect that has impact on the customer/patient. The most obvious is the impact on manufacturing (e.g medical devices, chemical compounds) but also for any other process/workflow critical decisions are based on. 

## Basics that are not exclusively applicable to CSV

Going with good practices, any IT system will have 3 separate environments 

* *Development*: Small replica of the production system. No change control. Used for IT internal testing of new features/changes. 
* *Test/Staging/QA*: Small replica of the production system. Fully change controlled. Used to allow for business testing before rolling out the change to Production.
* *Production*: Day to day system for productive use. Change control for system-wide changes is in place.

> **_NOTE:_**  This distinction between the three environments (DEV/TEST/PROD) is solely  relevant for IT. It should not be confused by typical business workflows where code is promoted through various lifecycle states that are named very similar if not the same. While the IT environment names resemble a physically different environment, all business workflow environments live on a single IT environment. From a business perspective they should only ever be concerned about IT production where they can promote their deliverables through the various stages. Only exception to that rule if they do OQ/PQ testing.   

## What happens if a system is going through the CSV process ? 

The CSV process starts when a new system is needed and it is identified for CSV based on a *Risk Assessment*. Critical factors here to warrant a CSV process is the work on patient data, potential impact on the same and so on. Such a *Risk Assessment* is a collaboration between QA, Business and IT. Alongside the Risk assessment, *User Requirement Specifications (URS)* are drafted. Those contain all the requirements (business workflows, performance needs, ...). Typically those requirements are phrased in the form of user stories. The responsibility here is on the business side. 

Once the URS has been finalized, the IT team is producing *Design Specifications* where the technical design and the identified business needs are aligned. 

In some cases an additional *Solution Concept* is created that solely focuses on the technical aspects. 

As a last step before starting the build of the system, a so-called *Validation Plan* is created. This document describes the overall solution and how the system eventually is validated. It links to all the aforementioned documents but also includes all the actual steps that lead to the system becoming the "Validation" status. Those steps are more along the lines of actually doing things. They include two main steps: Qualification and Validation. 

Qualification ensures that the system is built in a fully reproducible manner (e.g. so that in the case of the system being destroyed it can be replicated in a consistent manner). 

Validation finally documents that the business processes can be executed successfully on the new system. 

Qualification splits into three steps.

* *Installation Qualification (IQ)*: Reproducible installation of the system ranging from setting up hardware, installing operating system and configuration of end user applications. Done exclusively by the IT team. 
* *Operational Qualification (OQ)*: Basic smoke tests (e.g. to ensure that the respective applications can be started). Executed by IT (sometimes with help from business)
* *Performance Qualification (PQ)*: Tests to ensure that the installed application(s) produce correct results in the business workflows identified in the URS. 

In order to trace the Qualification steps (especially PQ tests) to the URS, a so-called *Traceability Matrix* is created. 

Tools are qualified following the process above, business processes however are validated. 

The last statement sometimes leads to some confusion. Typically the PQ tests are tests of the tools but sometimes also used as an example to test business workflows which leaves a bit of a grey area. If one thinks about general Software Validation, some people think that doing enough tests of the software tools is enough to get the validation stamp. If and only if those tests can be linked back to URS elements however, the system gets validation status. 

At the very end and following successful execution of the Qualification Steps, a *Validation Report* is written and signed off by all parties. This step marks the first official release of the new system. 

Additionally a *Configuration Master File* is created that lists all the characteristics of the system (Hardware and software used including versions etc...)  

### Additional details to the IQ/OQ/PQ process

If a new system is setup or changes are being made to a CSV controlled system (see next section). The following qualification process happens:

1. IT is developing IQ and OQ scripts in their Development environment. In parallel business is planning for some PQ scripts.
2. Validation Plan (or the IAQS) is approved.
3. Same IQ and OQ scripts are run in the Test/QA environment by IT.
4. PQ testing by business users in Test/QA.
5. Interim Validation/Implementation Report is written detailing the results and any possible deviation.
6. Applying all changes to the production system (Typically only IQ and OQ run - PQ testing is sufficient to be done in Test/QA as IQ/OQ are reproducible.
7. Finalization of the Validation/Implementation Report concludes the Release/Change of the system.

## Changes to a CSV controlled system

Any change to a CSV controlled system triggers a "mini-validation". In general almost the same process applies but the *Validation Plan* is replaced by an *Impact Analysis and Quality Strategy (IAQS)* where the planned changes are documented including IQ/OQ/PQ, and both the impact and risk on the system including needed changes to CSV documents documented. 

Upon implementing all the changes, instead of a *Validation Report* an *Implementation Report (IR)* is created that summarizes the Change including any deviations etc... Once this is signed off, the modified system is ready for use. 





