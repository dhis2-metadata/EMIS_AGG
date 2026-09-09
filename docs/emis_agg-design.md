# SDG4 Education Toolkit { #emis-agg-design }

## 1. Introduction

The SDG 4 Education Toolkit provides a standardized digital toolkit for collecting, analysing, and reporting education data required to monitor Sustainable Development Goal 4 (SDG 4): Ensure inclusive and equitable quality education and promote lifelong learning opportunities for all.

The toolkit is implemented using the DHIS2 aggregate data model and provides a harmonized framework for routine education reporting from schools, districts, and national education authorities. It enables countries to collect a core set of education statistics while remaining flexible enough to accommodate national Education Management Information System (EMIS) requirements.

The toolkit combines:

1. Annual School Census (ASC) data.
2. District population datasets.
3. Education survey datasets.
4. National education expenditure datasets.

Together, these datasets provide the information required to monitor education access, participation, equity, quality, learning environment, teachers, ICT, and financing.

## 2. Background

Education systems require reliable, timely, and comparable data to support planning, budgeting, service delivery, and monitoring of national and global education commitments.

Many countries collect education statistics through annual school census exercises and separate surveys, but these datasets are often fragmented and difficult to analyse together. The SDG 4 Education Toolkit addresses this challenge by providing a unified metadata package that aligns routine education reporting with internationally recognised education indicators.

The toolkit is primarily aligned with:

* Sustainable Development Goal 4 (SDG 4).
* UNESCO Institute for Statistics (UIS) SDG 4 Indicator Framework.
* African Union Continental Education Strategy for Africa (CESA) where applicable.
* National EMIS reporting requirements.

Rather than replacing an existing EMIS, the toolkit provides a minimum harmonised reporting framework that countries can localise and extend according to national education policies and reporting needs.

## 3. Purpose

The purpose of the SDG 4 Education Toolkit is to provide DHIS2 implementers and Ministries of Education with a reusable metadata package that supports routine collection and analysis of SDG 4 education indicators.

The toolkit aims to:

1. Standardize education data collection across administrative levels.
2. Collect the minimum routine education data required for SDG 4 reporting
3. Harmonize routine education reporting.
4. Improve education planning and resource allocation.
5. Strengthen monitoring of education equity and inclusion.
6. upport reporting to UNESCO Institute for Statistics (UIS), SDG 4, and regional education frameworks such as CESA.

## 4. Toolkit Scope

The toolkit supports data collection across three administrative levels.
| Administrative level | Purpose                                                            |
| -------------------- | ------------------------------------------------------------------ |
| **School**           | Routine annual School Census reporting for education institutions. |
| **District**         | Population denominators and education survey reporting.            |
| **National**         | Education expenditure and financing information.                   |

Core design principles
* Harmonized metadata - Common education data elements aligned with international standards.
* Scalable architecture - Supports national, provincial, district, and school reporting structures.
* Configurable implementation - Countries may add or remove metadata according to national policies.
* Analytics - Metadata is designed so indicators and dashboards work immediately after installation.
* SDG Alignment - Data elements are selected because they contribute to SDG 4 monitoring.

## 5. System Design Overview

**Reporting Levels**

| Level | Primary Reporting Unit |
|---|---|
| **School** | Schools submit Annual School Census datasets. |
| **District** | District offices submit population and survey datasets and validate school submissions. |
| **National** | Ministry of Education reports national expenditure and analyses national indicators. |

**DHIS2 Configuration Components**

| Metadata Component | Purpose |
|---|---|
| **Datasets** | Annual reporting forms. ||
| **Data Elements** | Individual education variables. |
| **Category Combinations** | Age, sex, disability, and grade disaggregation. |
| **Indicators** | SDG 4 calculations. |
| **Dashboards** | Visual monitoring. |

## 6. Toolkit Structure

The Toolkit Contains Eight Datasets

| Dataset | Frequency | Reporting Level |
|---|---|---|
| EMIS Toolkit - ECD (Early Childhood)| Annual | School |
| EMIS Toolkit - Primary | Annual | School |
| EMIS Toolkit - Lower Secondary | Annual | School |
| EMIS Toolkit - Upper Secondary Annual | Annual | School |
| EMIS Toolkit - Tertiary | Annual | Institution |
| Population Dataset | Annual | District |
| EMIS Toolkit - Household Survey| Annual / Survey Cycle | District |
|EMIS Toolkit - National Expenditure** | Annual | National |

**Dataset Relationships**

1. School datasets produce routine education statistics.
2. District datasets provide denominators and contextual indicators.
3. National expenditure supports financing indicators.

## 7. SDG4 Indicator framework

SDG4 Targets covered

| Target Area | Focus |
|---|---|
| **4.1** | Primary and secondary education |
| **4.2** | Early childhood development |
| **4.3** | Technical, vocational, and tertiary education |
| **4.4** | ICT and relevant skills |
| **4.5** | Equity and inclusion |
| **4.6** | Youth and adult literacy |
| **4.a** | Education facilities |
| **4.c** | Teachers |


## 8.Analytics and Dashboard

The SDG4 Education Toolkit includes dashboards that bring together data from the Annual School Census, household surveys, population estimates and other relevant sources to support SDG4 monitoring and reporting.

The dashboards provide:

1. SDG4 indicator monitoring across key education targets.
2. National and subnational analysis by region/district.
3. Disaggregation by sex, education level, grade and institution ownership where available.
4. Trend analysis to monitor changes over time.
5. Equity analysis to identify differences between population groups.
6. Benchmark comparison to assess progress against national targets.
7. Visualisations using charts, maps, scorecards and tables.

Key analytics include:

1. Enrolment and completion rates.
2. Learning proficiency in reading and mathematics.
3. Out-of-school and over-age-for-grade rates.
4. ECD participation and developmental outcomes.
5. Youth and adult literacy and numeracy.
6. Availability of basic school services and infrastructure.
7. Teacher availability and pupil-trained teacher ratios.

The dashboard is intended to provide decision-makers with a consolidated view of education system performance and support evidence-based planning and SDG4 reporting.

![](images/sdg4.png)

## 9.Intended Users
| User | Primary Responsibilities |
|---|---|
| **School Administrators** | Capture Annual School Census data. |
| **District Education Officers** | Validate school submissions and analyse district indicators. |
| **Provincial Education Teams** | Monitor regional performance and support planning. |
| **Ministry of Education** | National reporting, dashboards, and SDG monitoring. |
| **Planning Departments** | Budget allocation and resource planning. |
| **Development Partners** | Programme monitoring and reporting. |

User groups

| User Group         | Dashboard       | Program Metadata   | Program Data        |
|--------------------|-----------------|---------------------|----------------------|
| **EMIS-Admin**   | Can edit and view | Can edit and view   | No access            |
| **EMIS-Access**  | Can edit and view | Can view only       | Can view only        |
| **EMIS-Data capture** | No access       | Can view only       | Can capture and view |


