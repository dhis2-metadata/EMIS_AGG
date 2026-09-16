# Education Toolkit Installation Guide { #emis-agg-installation }

> **Draft for review.** Points marked **[confirm]** need checking before publication.

## 1. Overview

The SDG 4 Education Toolkit is distributed as a set of DHIS2 metadata packages. Each `.json` file
contains a `package` component recording the package version, the DHIS2 version it was built for,
and its contents.

This guide covers three installation scenarios:

1. **A new DHIS2 instance**, with no existing education metadata.
2. **An instance that already holds metadata**, where conflicts and duplication have to be resolved.
3. **The SDG 4 dashboard alone**, mapped onto education data the instance already collects.

Read the whole guide before starting. Identify your scenario in section 1.2, and follow that
chapter together with the configuration steps in section 6, which apply to all three.

> **Warning**
>
> Install into a test or staging instance first, and only then into production. This is not a
> formality: a single unresolvable reference can cause DHIS2 to reject an entire package, and a
> package installed in the wrong order can overwrite indicator definitions that took a day to
> configure.

### 1.1 The package files

| File | Data collection | Indicators | Dashboard and visualisations |
|---|---|---|---|
| **COMPLETE metadata package** | All seven data sets | All, configured | **Yes** |
| **SDG4 Dashboard package** | None | The 50 SDG 4 indicators, **unconfigured** | **Yes** |
| **ECD Tool** | ECD data set | Its own, configured | No |
| **Primary Tool** | Primary data set | Its own, configured | No |
| **Lower Secondary Tool** | Lower Secondary data set | Its own, configured | No |
| **Upper Secondary Tool** | Upper Secondary data set | Its own, configured | No |
| **Tertiary Tool** | Tertiary data set | Its own, configured | No |
| **Household Survey** | Household Survey data set | Its own, configured | No |
| **National Expenditure** | National Expenditure data set | Its own, configured | No |

Each tool package contains its data set, sections, data entry form, data element group, data
elements, indicator group and indicators.

Every file also carries the three toolkit user groups, the package owner account and its user role
(see section 6.6).

### 1.2 Choosing a path

**The data set packages and the SDG 4 Dashboard package are alternatives. Do not install both.**

The Dashboard package contains the same 50 indicators as the data set packages, under the same
identifiers, but with their numerators and denominators left unconfigured. Installing one over the
other replaces the definitions:

- Installing the Dashboard package onto an instance that already has any tool package **resets those
  indicators to an unconfigured state** — 23 of them for the Primary Tool alone, all 50 on a
  complete installation.
- Installing a tool package onto an instance where the Dashboard indicators have been mapped by hand
  **replaces that mapping** with the toolkit's own expressions, which reference data elements the
  instance may not have.

| Your situation | Install |
|---|---|
| Adopting the toolkit's data collection and wanting the SDG 4 dashboard | **COMPLETE package** |
| Adopting only part of the toolkit's data collection, and providing your own analytics | The individual tool packages you need |
| Keeping your own education data collection, and wanting SDG 4 analytics over it | **SDG4 Dashboard package** — see section 5 |

**Only the COMPLETE package provides both data collection and the dashboard.** The individual tool
packages install data sets and fully configured indicators, but no dashboard and no visualisations —
those would have to be built locally.

The SDG4 Dashboard package cannot be used to supply them afterwards. It contains the same indicators
under the same identifiers, so importing it onto an instance that already has a tool package would
replace those working indicator definitions with unconfigured ones. If you want the toolkit's data
collection **and** the reference dashboard, install the COMPLETE package.

The individual tool packages can be combined freely with each other. Installing Primary and Lower
Secondary, and adding Upper Secondary later, is entirely supported; the packages share metadata and
the later import updates what the earlier one created rather than duplicating it.

### 1.3 SDG 4 indicators and the packages that provide them

Each data set package installs the SDG 4 indicators that its own data elements can compute.

| Package | SDG 4 indicators it provides |
|---|---|
| ECD | 11 |
| Primary | 23 |
| Lower Secondary | 17 |
| Upper Secondary | 11 |
| Tertiary | 6 |
| Household Survey | 13 |
| National Expenditure | 2 |

The counts total more than fifty because fifteen indicators are provided by more than one package.
They are the same objects, not duplicates: the six school-services indicators are built on
infrastructure data elements that every level collects, and the nine proficiency indicators can be
computed either from a level data set or from the household survey.

**The reference dashboard is only complete when all seven data set packages are installed.** Each
package brings the indicators it can compute; install a subset and the visualisations that depend on
the missing packages remain empty. A partial installation therefore degrades gradually rather than
failing — install only Primary and you still get all six target 4.a indicators and the primary
proficiency measures, but not the secondary completion and intake set, the early childhood
indicators, the youth and adult literacy measures, or the two finance indicators.

Appendix A lists every indicator in the toolkit and the package that provides it.

### 1.4 Requirements

**Install and upgrade as a superuser** — an account with the `ALL` authority. An account with
metadata import rights alone is not sufficient, for four reasons:

- Creating indicators, data elements and category options each require the corresponding per-type
  authority; metadata import authority on its own does not grant them.
- Setting public sharing requires the public-add authority for each object type. Without it the
  objects still import, with their sharing silently downgraded.
- On an **upgrade**, the package grants edit rights to the EMIS Admin group and read-only access to
  everyone else. A non-superuser outside that group cannot update the objects being replaced.
- Creating the package owner account requires user administration rights.

The package is built for DHIS2 **2.41.10**. **[confirm]** the minimum supported version.

Securing the server and the DHIS2 application is outside the scope of this document; see the
[DHIS2 documentation](https://docs.dhis2.org/).

## 2. Before you import

### 2.1 Default data dimensions

In early versions of DHIS2 the identifiers of the default data dimensions were generated per
instance. Later versions use fixed identifiers, and the configuration packages use those. If your
instance predates that change, its defaults will differ from the package's and the import will
conflict.

Check the four objects below and, where an identifier differs, search and replace it throughout the
`.json` file with the one from your instance.

| Object | Identifier in the package | API endpoint |
|---|---|---|
| Category | `GLevLNI9wkl` | `../api/categories.json?filter=name:eq:default` |
| Category option | `xYerKDKCefk` | `../api/categoryOptions.json?filter=name:eq:default` |
| Category combination | `bjDvmb4bfuf` | `../api/categoryCombos.json?filter=name:eq:default` |
| Category option combination | `HllvX50cXC0` | `../api/categoryOptionCombos.json?filter=name:eq:default` |

### 2.2 Organisation units

The package contains no organisation units and no references to any. The dashboard's visualisations
are configured to follow the organisation unit of whoever is viewing them, so no root organisation
unit identifier has to be substituted before import.

### 2.3 Always dry run first

Use the **Import/Export** app, or `POST /api/metadata`, with the dry run option enabled. A dry run
validates the whole payload and reports every problem without writing anything.

Pay attention to the *report status* rather than the HTTP response alone. An import that reports
`ERROR` has written nothing at all: DHIS2 aborts the entire payload when any object fails
validation, so a single unresolvable reference on a single object will cause several hundred objects
to be ignored. The object reports in the response name the object and the reference at fault.

## 3. Scenario 1 — a new instance

With no existing metadata there is nothing to conflict with.

1. Complete section 2.1 if your instance predates fixed default identifiers.
2. Dry run the package. Expect no errors.
3. Import.
4. Go to section 6, **Configuration**.

## 4. Scenario 2 — an instance with existing metadata

### 4.1 Dry run and read the report

The most common conflict is that an object in the package has a name, short name or code already
used by an object in your database. DHIS2 reports these as validation errors naming both objects.

### 4.2 Conflicts to expect with this package

**Generic short names.** Short name uniqueness in DHIS2 is instance-wide. The toolkit collects
several concepts whose natural short names are generic — computers, electricity, internet, drinking
water, population estimates — and these collide readily with unrelated metadata. The toolkit's own
short names carry an ` (EMIS)` suffix for this reason, but a collision is still possible.

**Sex, and the options Male and Female.** Almost every instance already has these. This is the
clearest case for reusing existing metadata rather than duplicating it, and also the most
consequential — see the warning in section 4.4.

**Grade and age categories.** Frequently already present under national names.

### 4.3 Three ways to resolve a conflict

**Rename the existing object in DHIS2.** No change to the `.json` file, so the package remains
standard and future updates apply cleanly. Changes are made through the user interface, which is
less error-prone. Consider what else refers to the existing object — training material, standard
operating procedures, data dictionaries.

**Rename the object in the `.json` file.** Existing metadata is left untouched, which matters when
users, documentation or integrations depend on it. The package then differs from the standard
distribution, which has to be accounted for at upgrade.

For both, the change can be as small as a prefix or suffix.

**Reuse the existing metadata.** Remove the object from the `.json` file and replace every reference
to its identifier with the identifier already in the database. This avoids duplication entirely, but
it requires detailed knowledge of the DHIS2 metadata model, does not work for all object types, and
complicates future updates.

### 4.4 A warning about reusing disaggregations

Reusing an existing category or category combination is the most attractive form of the third
option and the most hazardous. Changing which options belong to a category causes DHIS2 to
regenerate that category's option combinations, and the toolkit's data entry forms address every
input by data element and category option combination. Regenerating combinations silently breaks
those forms: the fields remain on screen but no longer save to the intended place.

If you reuse a disaggregation, plan to review and repair the affected data entry forms.

### 4.5 Duplicate metadata after a clean import

An import can succeed without conflicts and still leave duplicates — data elements or option sets
that express the same concept as something already present. Resolving them is not only a matter of
tidying the database; existing objects may be referenced by integrations, training material and
standard operating procedures. Where duplicates are tolerable, sharing can hide the unused ones from
most users.

## 5. Scenario 3 — the SDG 4 dashboard alone

The Dashboard package is for an instance that already collects education data and wants SDG 4
analytics over it, without adopting the toolkit's own data sets.

### 5.1 What arrives, and what does not

The package installs the dashboard, its visualisations, the SDG 4 indicator group and 50 indicators.
Each indicator arrives with its name, its numerator description and its denominator description —
but with the numerator and denominator formulas left unconfigured.

Nothing has to be recreated. The dashboard layout, the visualisations and the indicator definitions
are all in place. The work is limited to configuring each indicator to read your instance's data.

### 5.2 Mapping the indicators

For each indicator in the SDG 4 indicator group:

1. Open the indicator in the **Maintenance** app.
2. Read the **numerator description** and **denominator description**. These state in words what the
   indicator requires — for example, the number of learners in the first grade of primary education,
   or the population of the corresponding single age year.
3. Identify the data elements in your instance that hold those figures, and the disaggregations they
   use.
4. Build the numerator and denominator expressions against them.
5. Save.

Two things to watch. **An unmapped indicator does not report an error** — it returns a constant
value. A dashboard that renders is therefore not evidence that mapping is complete. And **age bands
must match**: where an indicator expects a population denominator for a specific single year or age
range, a denominator covering a different range will produce a plausible-looking number rather than
a failure.

Work through the indicator group systematically and keep a record of what each was mapped to.

### 5.3 Upgrades will overwrite your mapping

The indicators installed by the Dashboard package keep their identifiers across versions. A later
release of the Dashboard package will therefore replace every indicator you mapped with the
unconfigured version again.

Once you have mapped the dashboard, treat the package as branched: record your expressions outside
DHIS2, and treat any future Dashboard package release as something to compare and merge rather than
import directly.

## 6. Configuration

These steps apply to all three scenarios.

### 6.1 Assign data sets to organisation units

A data set is only available for data entry at the organisation units it is assigned to.

| Data set | Assign to |
|---|---|
| ECD, Primary, Lower Secondary, Upper Secondary | Schools |
| Tertiary | Tertiary institutions |
| Household Survey | Districts |
| National Expenditure | The national organisation unit |

**The toolkit expects each education level to be represented by its own organisation unit.** Under
that arrangement each data set has its own reporting unit and no value is shared between levels.

This matters because the level data sets deliberately share data elements. Of the 42 data elements
in each of the Primary, Lower Secondary and Upper Secondary data sets, **28 are common to all
three** — the facilities, water and sanitation, furniture and ICT sections, the headteacher, and the
disability measure. None of them is disaggregated by education level, because they describe the
institution rather than the level: a school has one electricity supply and one set of classrooms.

Where a single organisation unit is assigned more than one level data set — a school running primary
through upper secondary as one unit — those 28 fields become a single value for that organisation
unit and reporting period, visible in every form, with the most recent entry replacing the previous
one. A clerk completing the lower secondary form and entering the number of classrooms in the
secondary block will overwrite the figure the primary form recorded for the whole school.

**Such institutions must be restructured into level-specific organisation units before the data sets
are assigned.** A combined school becomes a parent organisation unit with a child for each level it
offers, and each level data set is assigned to its own child.

### 6.2 Add users to the toolkit user groups

The package installs three user groups, with no members. Assigning users to them is part of
installation.

| User group | Dashboard and visualisations | Metadata | Data |
|---|---|---|---|
| **EMIS Admin** | View and edit | View and edit | No access |
| **EMIS Access** | View | View | View |
| **EMIS Data Capture** | View | View | Capture and view |

**The dashboard is not publicly visible.** A user who belongs to none of these groups will not see
it at all. Adding users to a group is therefore one of the first steps after import, not an
optional refinement.

### 6.3 User roles

Users need a user role to reach the relevant applications. Two are typically sufficient:

1. **Data capture** — add and edit data values, and access the Data Entry app.
2. **Data analysis** — access dashboards, Data Visualizer, pivot tables, maps and reports.

See the DHIS2 documentation on configuring user roles.

### 6.4 Populate the Master School List

The package installs an organisation unit group named **EMIS - Master School List**, which ships
empty.

It is the denominator of the six SDG 4.a indicators that express school services as a proportion of
all schools. Until it contains the country's schools, at the organisation unit level where the data
sets are captured, those six indicators return no value and no error.

Populate it through the **Maintenance** app or the API, and keep it current as schools are added to
the hierarchy.

### 6.5 Connect population data

Several indicators — gross enrolment ratios, out-of-school rates and the proficiency rates — need a
population denominator by age group and sex.

The toolkit does not include a population data set, on the assumption that the target system already
maintains population estimates. It supplies the population data element itself. There are three
ways to connect it:

1. **Add the toolkit's population data element to a data set.** Either an existing population data
   set, or one created for the purpose, assigned to the organisation units that report denominators.
2. **Use the toolkit's data element and import the values through the API.** No data entry form is
   needed; the data element only has to belong to a data set assigned to the relevant organisation
   units.
3. **Use the population data elements already in your instance.** This requires editing the
   toolkit's indicators to reference them.

If you take the third route, **check that the age groups align.** The toolkit's population data
element is disaggregated by population age group and sex, and each indicator selects specific age
bands from it. An existing population data element organised into different bands will produce a
number rather than an error, and the resulting rates will be quietly wrong.

Until population data is connected, the indicators that depend on it return no value.

### 6.6 The package owner account

Every package file contains a disabled user account named `package_admin` and a user role named
`Package User Role`.

The account exists so that the package's metadata has a consistent owner on any instance. It is
disabled, has no password, and nobody logs in as it. It requires no configuration, and it appears in
the Users app only if an organisation unit is assigned to it, which is not necessary.

### 6.7 Sharing

Sharing is pre-configured for dashboards, visualisations, data sets, data elements, indicators,
categories and category options, as described in section 6.2. Adjust it to your own arrangements
where needed; see the DHIS2 documentation on sharing.

## 7. Verifying the installation

Work through this sequence before handing the instance over.

1. The expected data sets appear in the **Data Entry** app for an organisation unit you assigned
   them to.
2. Opening a data set renders its form, with all its sections.
3. A value can be entered and saved.
4. The dashboard is visible to a user in one of the toolkit user groups.
5. Once analytics has run, the dashboard shows figures rather than empty charts.
6. Spot-check one indicator that depends on population data and one that depends on the Master
   School List. Both returning values confirms that sections 6.4 and 6.5 are complete.

## 8. Adapting the toolkit

Local adaptations are expected. Common ones:

- Adding national data elements to a form.
- Renaming data elements and category options to national conventions.
- Adding or adjusting translations.
- Adding indicators for national reporting.

Two cautions. Adding or removing a data element means updating **both** the section and the custom
data entry form; the custom forms are supplied for reference and can also be replaced by section
forms entirely. And changing a category or category combination regenerates its category option
combinations, which breaks the custom forms — see section 4.4.

Where possible, extend the toolkit alongside its own metadata rather than editing shipped objects in
place. That distinction is what makes upgrades manageable.

## 9. Upgrading to a later version

A later package version replaces any object whose identifier matches, so local changes to shipped
objects do not survive an upgrade.

**Once you branch, maintain the branch.** If you have modified shipped objects, a straightforward
import of the next release will overwrite that work. Record what you changed, compare it against the
new release, and merge deliberately.

Take a database backup and dry run the new version before importing it, as for a first installation.

## 10. Removing metadata

Removing metadata you do not intend to use keeps the instance clean and avoids confusing users.

Remove objects in dependency order — indicators before the data elements they reference, sections
and forms before their data sets — and expect DHIS2 to refuse a deletion where something still
refers to the object. The error message names the dependency.

Removing metadata from a live instance requires care and a good understanding of DHIS2
dependencies. Where an object is unused but awkward to remove, sharing can hide it instead.

## Appendix A — Indicators provided by each package

Every indicator in the toolkit, and the package or packages that install it. An indicator listed
against more than one package is the **same object** in each: installing a second package that
contains it updates the object rather than creating another.

The SDG 4 column marks membership of the SDG 4 indicator group, which is what the reference
dashboard draws on.

| Indicator | Code | Provided by | SDG 4 |
|---|---|---|---|
| Adults aged 25 and over achieving at least a minimum proficiency level in mathematics | `SDG4_ADULT_MPL_MATHS` | Household Survey | SDG 4 |
| Adults aged 25 and over achieving at least a minimum proficiency level in reading | `SDG4_ADULT_MPL_READING` | Household Survey | SDG 4 |
| Books, subject 1 (ECD) | `EMIS_ECD_BOOKS_SUBJECT_1` | ECD |  |
| Books, subject 1 (Lower Secondary) | `EMIS_LSEC_BOOKS_SUBJECT_1` | Lower Secondary |  |
| Books, subject 1 (Primary) | `EMIS_PRI_BOOKS_SUBJECT_1` | Primary |  |
| Books, subject 1 (Upper Secondary) | `EMIS_USEC_BOOKS_SUBJECT_1` | Upper Secondary |  |
| Books, subject 2 (ECD) | `EMIS_ECD_BOOKS_SUBJECT_2` | ECD |  |
| Books, subject 2 (Lower Secondary) | `EMIS_LSEC_BOOKS_SUBJECT_2` | Lower Secondary |  |
| Books, subject 2 (Primary) | `EMIS_PRI_BOOKS_SUBJECT_2` | Primary |  |
| Books, subject 2 (Upper Secondary) | `EMIS_USEC_BOOKS_SUBJECT_2` | Upper Secondary |  |
| Books, subject 3 (ECD) | `EMIS_ECD_BOOKS_SUBJECT_3` | ECD |  |
| Books, subject 3 (Lower Secondary) | `EMIS_LSEC_BOOKS_SUBJECT_3` | Lower Secondary |  |
| Books, subject 3 (Primary) | `EMIS_PRI_BOOKS_SUBJECT_3` | Primary |  |
| Books, subject 3 (Upper Secondary) | `EMIS_USEC_BOOKS_SUBJECT_3` | Upper Secondary |  |
| Books, subject 4 (ECD) | `EMIS_ECD_BOOKS_SUBJECT_4` | ECD |  |
| Books, subject 4 (Lower Secondary) | `EMIS_LSEC_BOOKS_SUBJECT_4` | Lower Secondary |  |
| Books, subject 4 (Primary) | `EMIS_PRI_BOOKS_SUBJECT_4` | Primary |  |
| Books, subject 4 (Upper Secondary) | `EMIS_USEC_BOOKS_SUBJECT_4` | Upper Secondary |  |
| Certified teachers (Lower Secondary) | `EMIS_LSEC_TEACHERS_CERTIFIED` | Lower Secondary |  |
| Certified teachers (Upper Secondary) | `EMIS_USEC_TEACHERS_CERTIFIED` | Upper Secondary |  |
| Children achieving at least a minimum proficiency level in mathematics, end of cycle (Primary) | `SDG4_PRI_MPL_MATHS_END` | Primary, Household Survey | SDG 4 |
| Children achieving at least a minimum proficiency level in mathematics, mid-cycle (Primary) | `SDG4_PRI_MPL_MATHS_MID` | Primary, Household Survey | SDG 4 |
| Children achieving at least a minimum proficiency level in reading, end of cycle (Primary) | `SDG4_PRI_MPL_READING_END` | Primary, Household Survey | SDG 4 |
| Children achieving at least a minimum proficiency level in reading, mid-cycle (Primary) | `SDG4_PRI_MPL_READING_MID` | Primary, Household Survey | SDG 4 |
| Children achieving at least a minimum proficiency level in science, end of cycle (Primary) | `SDG4_PRI_MPL_SCIENCE_END` | Primary, Household Survey | SDG 4 |
| Children achieving at least a minimum proficiency level in science, mid-cycle (Primary) | `SDG4_PRI_MPL_SCIENCE_MID` | Primary, Household Survey | SDG 4 |
| Children achieving at least a minimum proficiency level, end of cycle (Primary) | `EMIS_PRI_MPL_END` | Primary, Household Survey |  |
| Children achieving at least a minimum proficiency level, mid-cycle (Primary) | `EMIS_PRI_MPL_MID` | Primary, Household Survey |  |
| Completion rate (Lower Secondary) | `SDG4_LSEC_COMPLETION_RATE` | Lower Secondary | SDG 4 |
| Completion rate (Primary) | `SDG4_PRI_COMPLETION_RATE` | Primary | SDG 4 |
| Completion rate (Tertiary) | `EMIS_TERT_COMPLETION_RATE` | Tertiary |  |
| Completion rate (Upper Secondary) | `SDG4_USEC_COMPLETION_RATE` | Upper Secondary | SDG 4 |
| Enrolment by nationality, citizens (ECD) | `EMIS_ECD_NATIONALITY_CITIZENS` | ECD |  |
| Enrolment by nationality, citizens (Lower Secondary) | `EMIS_LSEC_NATIONALITY_CITIZENS` | Lower Secondary |  |
| Enrolment by nationality, citizens (Primary) | `EMIS_PRI_NATIONALITY_CITIZENS` | Primary |  |
| Enrolment by nationality, citizens (Upper Secondary) | `EMIS_USEC_NATIONALITY_CITIZENS` | Upper Secondary |  |
| Enrolment by nationality, non-refugee foreigners (ECD) | `EMIS_ECD_NATIONALITY_NON_REFUGEES` | ECD |  |
| Enrolment by nationality, non-refugee foreigners (Lower Secondary) | `EMIS_LSEC_NATIONALITY_NON_REFUGEES` | Lower Secondary |  |
| Enrolment by nationality, non-refugee foreigners (Primary) | `EMIS_PRI_NATIONALITY_NON_REFUGEES` | Primary |  |
| Enrolment by nationality, non-refugee foreigners (Upper Secondary) | `EMIS_USEC_NATIONALITY_NON_REFUGEES` | Upper Secondary |  |
| Enrolment by nationality, refugees (ECD) | `EMIS_ECD_NATIONALITY_REFUGEES` | ECD |  |
| Enrolment by nationality, refugees (Lower Secondary) | `EMIS_LSEC_NATIONALITY_REFUGEES` | Lower Secondary |  |
| Enrolment by nationality, refugees (Primary) | `EMIS_PRI_NATIONALITY_REFUGEES` | Primary |  |
| Enrolment by nationality, refugees (Upper Secondary) | `EMIS_USEC_NATIONALITY_REFUGEES` | Upper Secondary |  |
| Furniture - book shelves | `EMIS_SHARED_FURNITURE_BOOKSHELVES` | ECD, Primary, Lower Secondary, Upper Secondary |  |
| Furniture - learner desks | `EMIS_SHARED_FURNITURE_LEARNER_DESKS` | ECD, Primary, Lower Secondary, Upper Secondary |  |
| Furniture - staff desks | `EMIS_SHARED_FURNITURE_STAFF_DESKS` | ECD, Primary, Lower Secondary, Upper Secondary |  |
| Furniture - white boards | `EMIS_SHARED_FURNITURE_WHITEBOARDS` | ECD, Primary, Lower Secondary, Upper Secondary |  |
| Gross enrolment ratio (ECD) | `SDG4_ECD_GER` | ECD | SDG 4 |
| Gross enrolment ratio (Lower Secondary) | `EMIS_LSEC_GER` | Lower Secondary |  |
| Gross enrolment ratio (Primary) | `EMIS_PRI_GER` | Primary |  |
| Gross enrolment ratio (Tertiary) | `EMIS_TERT_GER` | Tertiary |  |
| Gross enrolment ratio (Upper Secondary) | `SDG4_USEC_GER` | Upper Secondary | SDG 4 |
| Gross intake ratio (Lower Secondary) | `SDG4_LSEC_GROSS_INTAKE_RATIO` | Lower Secondary | SDG 4 |
| Gross intake ratio (Primary) | `EMIS_PRI_GROSS_INTAKE_RATIO` | Primary |  |
| Gross intake ratio (Upper Secondary) | `SDG4_USEC_GROSS_INTAKE_RATIO` | Upper Secondary | SDG 4 |
| Learners developmentally on track (ECD) | `SDG4_ECD_DEV_ON_TRACK` | ECD | SDG 4 |
| Learners in the reporting year, aged 10 (Primary) | `EMIS_PRI_LEARNERS_AGE_10` | Primary |  |
| Learners in the reporting year, aged 11 (Primary) | `EMIS_PRI_LEARNERS_AGE_11` | Primary |  |
| Learners in the reporting year, aged 12 (Lower Secondary) | `EMIS_LSEC_LEARNERS_AGE_12` | Lower Secondary |  |
| Learners in the reporting year, aged 12 (Primary) | `EMIS_PRI_LEARNERS_AGE_12` | Primary |  |
| Learners in the reporting year, aged 13 (Lower Secondary) | `EMIS_LSEC_LEARNERS_AGE_13` | Lower Secondary |  |
| Learners in the reporting year, aged 13 (Primary) | `EMIS_PRI_LEARNERS_AGE_13` | Primary |  |
| Learners in the reporting year, aged 14 (Lower Secondary) | `EMIS_LSEC_LEARNERS_AGE_14` | Lower Secondary |  |
| Learners in the reporting year, aged 15 (Lower Secondary) | `EMIS_LSEC_LEARNERS_AGE_15` | Lower Secondary |  |
| Learners in the reporting year, aged 16 (Lower Secondary) | `EMIS_LSEC_LEARNERS_AGE_16` | Lower Secondary |  |
| Learners in the reporting year, aged 16 (Upper Secondary) | `EMIS_USEC_LEARNERS_AGE_16` | Upper Secondary |  |
| Learners in the reporting year, aged 17 (Lower Secondary) | `EMIS_LSEC_LEARNERS_AGE_17` | Lower Secondary |  |
| Learners in the reporting year, aged 17 (Upper Secondary) | `EMIS_USEC_LEARNERS_AGE_17` | Upper Secondary |  |
| Learners in the reporting year, aged 18 (Upper Secondary) | `EMIS_USEC_LEARNERS_AGE_18` | Upper Secondary |  |
| Learners in the reporting year, aged 3 (ECD) | `EMIS_ECD_LEARNERS_AGE_3` | ECD |  |
| Learners in the reporting year, aged 4 (ECD) | `EMIS_ECD_LEARNERS_AGE_4` | ECD |  |
| Learners in the reporting year, aged 5 (ECD) | `EMIS_ECD_LEARNERS_AGE_5` | ECD |  |
| Learners in the reporting year, aged 6 (ECD) | `EMIS_ECD_LEARNERS_AGE_6` | ECD |  |
| Learners in the reporting year, aged 6 (Primary) | `EMIS_PRI_LEARNERS_AGE_6` | Primary |  |
| Learners in the reporting year, aged 7 (Primary) | `EMIS_PRI_LEARNERS_AGE_7` | Primary |  |
| Learners in the reporting year, aged 8 (Primary) | `EMIS_PRI_LEARNERS_AGE_8` | Primary |  |
| Learners in the reporting year, aged 9 (Primary) | `EMIS_PRI_LEARNERS_AGE_9` | Primary |  |
| Learners in the reporting year, aged over 13 (Primary) | `EMIS_PRI_LEARNERS_AGE_OVER_13` | Primary |  |
| Learners in the reporting year, aged over 17 (Lower Secondary) | `EMIS_LSEC_LEARNERS_AGE_OVER_17` | Lower Secondary |  |
| Learners in the reporting year, aged over 18 (Upper Secondary) | `EMIS_USEC_LEARNERS_AGE_OVER_18` | Upper Secondary |  |
| Learners in the reporting year, aged over 6 (ECD) | `EMIS_ECD_LEARNERS_AGE_OVER_6` | ECD |  |
| Learners in the reporting year, aged under 12 (Lower Secondary) | `EMIS_LSEC_LEARNERS_AGE_UNDER_12` | Lower Secondary |  |
| Learners in the reporting year, aged under 16 (Upper Secondary) | `EMIS_USEC_LEARNERS_AGE_UNDER_16` | Upper Secondary |  |
| Learners in the reporting year, aged under 3 (ECD) | `EMIS_ECD_LEARNERS_AGE_UNDER_3` | ECD |  |
| Learners in the reporting year, aged under 6 (Primary) | `EMIS_PRI_LEARNERS_AGE_UNDER_6` | Primary |  |
| Learners in the reporting year, level 1, female (ECD) | `EMIS_ECD_LEARNERS_LEVEL_1_FEMALE` | ECD |  |
| Learners in the reporting year, level 1, male (ECD) | `EMIS_ECD_LEARNERS_LEVEL_1_MALE` | ECD |  |
| Learners in the reporting year, level 2, female (ECD) | `EMIS_ECD_LEARNERS_LEVEL_2_FEMALE` | ECD |  |
| Learners in the reporting year, level 2, male (ECD) | `EMIS_ECD_LEARNERS_LEVEL_2_MALE` | ECD |  |
| Learners in the reporting year, level 3, female (ECD) | `EMIS_ECD_LEARNERS_LEVEL_3_FEMALE` | ECD |  |
| Learners in the reporting year, level 3, male (ECD) | `EMIS_ECD_LEARNERS_LEVEL_3_MALE` | ECD |  |
| Net enrolment rate (ECD) | `EMIS_ECD_NER` | ECD |  |
| Net enrolment rate (Lower Secondary) | `EMIS_LSEC_NER` | Lower Secondary |  |
| Net enrolment rate (Primary) | `EMIS_PRI_NER` | Primary |  |
| Net enrolment rate (Upper Secondary) | `EMIS_USEC_NER` | Upper Secondary |  |
| Number of teaching materials, subject 1 (ECD) | `EMIS_ECD_TEACHING_MATERIALS_SUBJECT_1` | ECD |  |
| Number of teaching materials, subject 1 (Lower Secondary) | `EMIS_LSEC_TEACHING_MATERIALS_SUBJECT_1` | Lower Secondary |  |
| Number of teaching materials, subject 1 (Primary) | `EMIS_PRI_TEACHING_MATERIALS_SUBJECT_1` | Primary |  |
| Number of teaching materials, subject 1 (Upper Secondary) | `EMIS_USEC_TEACHING_MATERIALS_SUBJECT_1` | Upper Secondary |  |
| Number of teaching materials, subject 2 (ECD) | `EMIS_ECD_TEACHING_MATERIALS_SUBJECT_2` | ECD |  |
| Number of teaching materials, subject 2 (Lower Secondary) | `EMIS_LSEC_TEACHING_MATERIALS_SUBJECT_2` | Lower Secondary |  |
| Number of teaching materials, subject 2 (Primary) | `EMIS_PRI_TEACHING_MATERIALS_SUBJECT_2` | Primary |  |
| Number of teaching materials, subject 2 (Upper Secondary) | `EMIS_USEC_TEACHING_MATERIALS_SUBJECT_2` | Upper Secondary |  |
| Number of teaching materials, subject 3 (ECD) | `EMIS_ECD_TEACHING_MATERIALS_SUBJECT_3` | ECD |  |
| Number of teaching materials, subject 3 (Lower Secondary) | `EMIS_LSEC_TEACHING_MATERIALS_SUBJECT_3` | Lower Secondary |  |
| Number of teaching materials, subject 3 (Primary) | `EMIS_PRI_TEACHING_MATERIALS_SUBJECT_3` | Primary |  |
| Number of teaching materials, subject 3 (Upper Secondary) | `EMIS_USEC_TEACHING_MATERIALS_SUBJECT_3` | Upper Secondary |  |
| Number of teaching materials, subject 4 (ECD) | `EMIS_ECD_TEACHING_MATERIALS_SUBJECT_4` | ECD |  |
| Number of teaching materials, subject 4 (Lower Secondary) | `EMIS_LSEC_TEACHING_MATERIALS_SUBJECT_4` | Lower Secondary |  |
| Number of teaching materials, subject 4 (Primary) | `EMIS_PRI_TEACHING_MATERIALS_SUBJECT_4` | Primary |  |
| Number of teaching materials, subject 4 (Upper Secondary) | `EMIS_USEC_TEACHING_MATERIALS_SUBJECT_4` | Upper Secondary |  |
| Out-of-school rate (Lower Secondary) | `SDG4_LSEC_OUT_OF_SCHOOL_RATE` | Lower Secondary | SDG 4 |
| Out-of-school rate (Primary) | `SDG4_PRI_OUT_OF_SCHOOL_RATE` | Primary | SDG 4 |
| Out-of-school rate (Upper Secondary) | `SDG4_USEC_OUT_OF_SCHOOL_RATE` | Upper Secondary | SDG 4 |
| Participation rate in organized learning, one year before the official primary entry age (ECD) | `SDG4_ECD_PARTICIPATION_ORGANIZED_LEARNING` | ECD | SDG 4 |
| Percentage distribution of public current expenditure on education - ECD | `EMIS_NAT_PCT_CURRENT_EXPENDITURE_ECD` | National Expenditure |  |
| Percentage distribution of public current expenditure on education - lower secondary | `EMIS_NAT_PCT_CURRENT_EXPENDITURE_LSEC` | National Expenditure |  |
| Percentage distribution of public current expenditure on education - primary | `EMIS_NAT_PCT_CURRENT_EXPENDITURE_PRI` | National Expenditure |  |
| Percentage distribution of public current expenditure on education - tertiary | `EMIS_NAT_PCT_CURRENT_EXPENDITURE_TERT` | National Expenditure |  |
| Percentage distribution of public current expenditure on education - upper secondary | `EMIS_NAT_PCT_CURRENT_EXPENDITURE_USEC` | National Expenditure |  |
| Percentage of children over-age for grade, level 1 (Lower Secondary) | `SDG4_LSEC_OVERAGE_L1` | Lower Secondary | SDG 4 |
| Percentage of children over-age for grade, level 1 (Primary) | `SDG4_PRI_OVERAGE_L1` | Primary | SDG 4 |
| Percentage of children over-age for grade, level 2 (Lower Secondary) | `SDG4_LSEC_OVERAGE_L2` | Lower Secondary | SDG 4 |
| Percentage of children over-age for grade, level 2 (Primary) | `SDG4_PRI_OVERAGE_L2` | Primary | SDG 4 |
| Percentage of children over-age for grade, level 3 (Lower Secondary) | `SDG4_LSEC_OVERAGE_L3` | Lower Secondary | SDG 4 |
| Percentage of children over-age for grade, level 3 (Primary) | `SDG4_PRI_OVERAGE_L3` | Primary | SDG 4 |
| Percentage of children over-age for grade, level 4 (Lower Secondary) | `SDG4_LSEC_OVERAGE_L4` | Lower Secondary | SDG 4 |
| Percentage of children over-age for grade, level 4 (Primary) | `SDG4_PRI_OVERAGE_L4` | Primary | SDG 4 |
| Percentage of children over-age for grade, level 5 (Primary) | `SDG4_PRI_OVERAGE_L5` | Primary | SDG 4 |
| Percentage of children over-age for grade, level 6 (Primary) | `SDG4_PRI_OVERAGE_L6` | Primary | SDG 4 |
| Percentage of children over-age for grade, level 7 (Primary) | `SDG4_PRI_OVERAGE_L7` | Primary | SDG 4 |
| Percentage of female teachers (ECD) | `EMIS_ECD_PCT_FEMALE_TEACHERS` | ECD |  |
| Percentage of female teachers (Lower Secondary) | `EMIS_LSEC_PCT_FEMALE_TEACHERS` | Lower Secondary |  |
| Percentage of female teachers (Primary) | `EMIS_PRI_PCT_FEMALE_TEACHERS` | Primary |  |
| Percentage of female teachers (Tertiary) | `EMIS_TERT_PCT_FEMALE_TEACHERS` | Tertiary |  |
| Percentage of female teachers (Upper Secondary) | `EMIS_USEC_PCT_FEMALE_TEACHERS` | Upper Secondary |  |
| Percentage of teachers qualified to teach according to national standards (ECD) | `EMIS_ECD_PCT_TEACHERS_NATIONAL_STANDARD` | ECD |  |
| Percentage of teachers qualified to teach according to national standards (Lower Secondary) | `EMIS_LSEC_PCT_TEACHERS_NATIONAL_STANDARD` | Lower Secondary |  |
| Percentage of teachers qualified to teach according to national standards (Primary) | `EMIS_PRI_PCT_TEACHERS_NATIONAL_STANDARD` | Primary |  |
| Percentage of teachers qualified to teach according to national standards (Tertiary) | `EMIS_TERT_PCT_TEACHERS_NATIONAL_STANDARD` | Tertiary |  |
| Percentage of teachers qualified to teach according to national standards (Upper Secondary) | `EMIS_USEC_PCT_TEACHERS_NATIONAL_STANDARD` | Upper Secondary |  |
| Percentage of teachers with the minimum required qualifications (ECD) | `SDG4_ECD_TEACH_MIN_QUAL` | ECD | SDG 4 |
| Percentage of teachers with the minimum required qualifications (Lower Secondary) | `SDG4_LSEC_TEACH_MIN_QUAL` | Lower Secondary | SDG 4 |
| Percentage of teachers with the minimum required qualifications (Primary) | `SDG4_PRI_TEACH_MIN_QUAL` | Primary | SDG 4 |
| Percentage of teachers with the minimum required qualifications (Tertiary) | `EMIS_TERT_TEACH_MIN_QUAL` | Tertiary |  |
| Percentage of teachers with the minimum required qualifications (Upper Secondary) | `SDG4_USEC_TEACH_MIN_QUAL` | Upper Secondary | SDG 4 |
| Proportion of children aged 24-59 months developmentally on track (ECD) | `SDG4_ECD_DEV_ON_TRACK_24_59M` | ECD | SDG 4 |
| Proportion of schools offering basic services - adapted infrastructure and materials for learners with disabilities | `SDG4_SCHOOLS_BASIC_SNE_INFRASTRUCTURE` | ECD, Primary, Lower Secondary, Upper Secondary, Tertiary | SDG 4 |
| Proportion of schools offering basic services - basic drinking water | `SDG4_SCHOOLS_BASIC_DRINKING_WATER` | ECD, Primary, Lower Secondary, Upper Secondary, Tertiary | SDG 4 |
| Proportion of schools offering basic services - basic handwashing facilities | `SDG4_SCHOOLS_BASIC_HANDWASHING` | ECD, Primary, Lower Secondary, Upper Secondary, Tertiary | SDG 4 |
| Proportion of schools offering basic services - computers for pedagogical purposes | `SDG4_SCHOOLS_BASIC_COMPUTERS` | ECD, Primary, Lower Secondary, Upper Secondary, Tertiary | SDG 4 |
| Proportion of schools offering basic services - electricity | `SDG4_SCHOOLS_BASIC_ELECTRICITY` | ECD, Primary, Lower Secondary, Upper Secondary, Tertiary | SDG 4 |
| Proportion of schools offering basic services - internet for pedagogical purposes | `SDG4_SCHOOLS_BASIC_INTERNET` | ECD, Primary, Lower Secondary, Upper Secondary, Tertiary | SDG 4 |
| Public expenditure on education as a percentage of gross national income | `SDG4_EXP_PCT_GNI` | National Expenditure | SDG 4 |
| Public expenditure on education as a percentage of total government expenditure | `SDG4_EXP_PCT_GOV_EXPENDITURE` | National Expenditure | SDG 4 |
| Pupil-latrine stance ratio (ECD) | `EMIS_ECD_PUPIL_LATRINE_RATIO` | ECD |  |
| Pupil-latrine stance ratio (Lower Secondary) | `EMIS_LSEC_PUPIL_LATRINE_RATIO` | Lower Secondary |  |
| Pupil-latrine stance ratio (Primary) | `EMIS_PRI_PUPIL_LATRINE_RATIO` | Primary |  |
| Pupil-latrine stance ratio (Upper Secondary) | `EMIS_USEC_PUPIL_LATRINE_RATIO` | Upper Secondary |  |
| Pupil-qualified teacher ratio (ECD) | `EMIS_ECD_PUPIL_QUALIFIED_TEACHER_RATIO` | ECD |  |
| Pupil-qualified teacher ratio (Lower Secondary) | `EMIS_LSEC_PUPIL_QUALIFIED_TEACHER_RATIO` | Lower Secondary |  |
| Pupil-qualified teacher ratio (Primary) | `EMIS_PRI_PUPIL_QUALIFIED_TEACHER_RATIO` | Primary |  |
| Pupil-qualified teacher ratio (Tertiary) | `EMIS_TERT_PUPIL_QUALIFIED_TEACHER_RATIO` | Tertiary |  |
| Pupil-qualified teacher ratio (Upper Secondary) | `EMIS_USEC_PUPIL_QUALIFIED_TEACHER_RATIO` | Upper Secondary |  |
| Pupil-teacher ratio (ECD) | `EMIS_ECD_PUPIL_TEACHER_RATIO` | ECD |  |
| Pupil-teacher ratio (Lower Secondary) | `EMIS_LSEC_PUPIL_TEACHER_RATIO` | Lower Secondary |  |
| Pupil-teacher ratio (Primary) | `EMIS_PRI_PUPIL_TEACHER_RATIO` | Primary |  |
| Pupil-teacher ratio (Tertiary) | `EMIS_TERT_PUPIL_TEACHER_RATIO` | Tertiary |  |
| Pupil-teacher ratio (Upper Secondary) | `EMIS_USEC_PUPIL_TEACHER_RATIO` | Upper Secondary |  |
| Pupil-trained teacher ratio (ECD) | `EMIS_ECD_PUPIL_TRAINED_TEACHER_RATIO` | ECD |  |
| Pupil-trained teacher ratio (Lower Secondary) | `EMIS_LSEC_PUPIL_TRAINED_TEACHER_RATIO` | Lower Secondary |  |
| Pupil-trained teacher ratio (Primary) | `SDG4_PRI_PUPIL_TRAINED_TEACHER_RATIO` | Primary | SDG 4 |
| Pupil-trained teacher ratio (Tertiary) | `EMIS_TERT_PUPIL_TRAINED_TEACHER_RATIO` | Tertiary |  |
| Pupil-trained teacher ratio (Upper Secondary) | `EMIS_USEC_PUPIL_TRAINED_TEACHER_RATIO` | Upper Secondary |  |
| Qualified teachers (Lower Secondary) | `EMIS_LSEC_TEACHERS_QUALIFIED` | Lower Secondary |  |
| Qualified teachers (Upper Secondary) | `EMIS_USEC_TEACHERS_QUALIFIED` | Upper Secondary |  |
| Repeaters in the reporting year, female (Lower Secondary) | `EMIS_LSEC_REPEATERS_FEMALE` | Lower Secondary |  |
| Repeaters in the reporting year, female (Upper Secondary) | `EMIS_USEC_REPEATERS_FEMALE` | Upper Secondary |  |
| Repeaters in the reporting year, level 1 (Primary) | `EMIS_PRI_REPEATERS_LEVEL_1` | Primary |  |
| Repeaters in the reporting year, level 2 (Primary) | `EMIS_PRI_REPEATERS_LEVEL_2` | Primary |  |
| Repeaters in the reporting year, level 3 (Primary) | `EMIS_PRI_REPEATERS_LEVEL_3` | Primary |  |
| Repeaters in the reporting year, level 4 (Primary) | `EMIS_PRI_REPEATERS_LEVEL_4` | Primary |  |
| Repeaters in the reporting year, level 5 (Primary) | `EMIS_PRI_REPEATERS_LEVEL_5` | Primary |  |
| Repeaters in the reporting year, level 6 (Primary) | `EMIS_PRI_REPEATERS_LEVEL_6` | Primary |  |
| Repeaters in the reporting year, level 7 (Primary) | `EMIS_PRI_REPEATERS_LEVEL_7` | Primary |  |
| Repeaters in the reporting year, male (Lower Secondary) | `EMIS_LSEC_REPEATERS_MALE` | Lower Secondary |  |
| Repeaters in the reporting year, male (Upper Secondary) | `EMIS_USEC_REPEATERS_MALE` | Upper Secondary |  |
| Repetition rate (Lower Secondary) | `EMIS_LSEC_REPETITION_RATE` | Lower Secondary |  |
| Repetition rate (Primary) | `EMIS_PRI_REPETITION_RATE` | Primary |  |
| Repetition rate (Tertiary) | `EMIS_TERT_REPETITION_RATE` | Tertiary |  |
| Repetition rate (Upper Secondary) | `EMIS_USEC_REPETITION_RATE` | Upper Secondary |  |
| School facilities - classrooms | `EMIS_SHARED_FACILITY_CLASSROOMS` | ECD, Primary, Lower Secondary, Upper Secondary |  |
| School facilities - classrooms with access to usable ramps | `EMIS_SHARED_FACILITY_CLASSROOMS_RAMPS` | ECD, Primary, Lower Secondary, Upper Secondary |  |
| School facilities - computer laboratory | `EMIS_SHARED_FACILITY_COMPUTER_LAB` | ECD, Primary, Lower Secondary, Upper Secondary |  |
| School facilities - dining hall | `EMIS_SHARED_FACILITY_DINING_HALL` | ECD, Primary, Lower Secondary, Upper Secondary |  |
| School facilities - kitchens | `EMIS_SHARED_FACILITY_KITCHENS` | ECD, Primary, Lower Secondary, Upper Secondary |  |
| School facilities - laboratories | `EMIS_SHARED_FACILITY_LABORATORIES` | Primary, Lower Secondary, Upper Secondary |  |
| School facilities - library | `EMIS_SHARED_FACILITY_LIBRARY` | ECD, Primary, Lower Secondary, Upper Secondary |  |
| School facilities - main hall | `EMIS_SHARED_FACILITY_MAIN_HALL` | ECD, Primary, Lower Secondary, Upper Secondary |  |
| School facilities - sickbay | `EMIS_SHARED_FACILITY_SICKBAY` | ECD, Primary, Lower Secondary, Upper Secondary |  |
| School facilities - staff quarters | `EMIS_SHARED_FACILITY_STAFF_QUARTERS` | ECD, Primary, Lower Secondary, Upper Secondary |  |
| School facilities - staff room | `EMIS_SHARED_FACILITY_STAFF_ROOM` | ECD, Primary, Lower Secondary, Upper Secondary |  |
| SNE learners, cognition (ECD) | `EMIS_ECD_SNE_LEARNERS_COGNITION` | ECD |  |
| SNE learners, cognition (Lower Secondary) | `EMIS_LSEC_SNE_LEARNERS_COGNITION` | Lower Secondary |  |
| SNE learners, cognition (Primary) | `EMIS_PRI_SNE_LEARNERS_COGNITION` | Primary |  |
| SNE learners, cognition (Tertiary) | `EMIS_TERT_SNE_LEARNERS_COGNITION` | Tertiary |  |
| SNE learners, cognition (Upper Secondary) | `EMIS_USEC_SNE_LEARNERS_COGNITION` | Upper Secondary |  |
| SNE learners, communication (ECD) | `EMIS_ECD_SNE_LEARNERS_COMMUNICATION` | ECD |  |
| SNE learners, communication (Lower Secondary) | `EMIS_LSEC_SNE_LEARNERS_COMMUNICATION` | Lower Secondary |  |
| SNE learners, communication (Primary) | `EMIS_PRI_SNE_LEARNERS_COMMUNICATION` | Primary |  |
| SNE learners, communication (Tertiary) | `EMIS_TERT_SNE_LEARNERS_COMMUNICATION` | Tertiary |  |
| SNE learners, communication (Upper Secondary) | `EMIS_USEC_SNE_LEARNERS_COMMUNICATION` | Upper Secondary |  |
| SNE learners, hearing (ECD) | `EMIS_ECD_SNE_LEARNERS_HEARING` | ECD |  |
| SNE learners, hearing (Lower Secondary) | `EMIS_LSEC_SNE_LEARNERS_HEARING` | Lower Secondary |  |
| SNE learners, hearing (Primary) | `EMIS_PRI_SNE_LEARNERS_HEARING` | Primary |  |
| SNE learners, hearing (Tertiary) | `EMIS_TERT_SNE_LEARNERS_HEARING` | Tertiary |  |
| SNE learners, hearing (Upper Secondary) | `EMIS_USEC_SNE_LEARNERS_HEARING` | Upper Secondary |  |
| SNE learners, mobility (ECD) | `EMIS_ECD_SNE_LEARNERS_MOBILITY` | ECD |  |
| SNE learners, mobility (Lower Secondary) | `EMIS_LSEC_SNE_LEARNERS_MOBILITY` | Lower Secondary |  |
| SNE learners, mobility (Primary) | `EMIS_PRI_SNE_LEARNERS_MOBILITY` | Primary |  |
| SNE learners, mobility (Tertiary) | `EMIS_TERT_SNE_LEARNERS_MOBILITY` | Tertiary |  |
| SNE learners, mobility (Upper Secondary) | `EMIS_USEC_SNE_LEARNERS_MOBILITY` | Upper Secondary |  |
| SNE learners, self-care (ECD) | `EMIS_ECD_SNE_LEARNERS_SELF_CARE` | ECD |  |
| SNE learners, self-care (Lower Secondary) | `EMIS_LSEC_SNE_LEARNERS_SELF_CARE` | Lower Secondary |  |
| SNE learners, self-care (Primary) | `EMIS_PRI_SNE_LEARNERS_SELF_CARE` | Primary |  |
| SNE learners, self-care (Tertiary) | `EMIS_TERT_SNE_LEARNERS_SELF_CARE` | Tertiary |  |
| SNE learners, self-care (Upper Secondary) | `EMIS_USEC_SNE_LEARNERS_SELF_CARE` | Upper Secondary |  |
| SNE learners, vision (ECD) | `EMIS_ECD_SNE_LEARNERS_VISION` | ECD |  |
| SNE learners, vision (Lower Secondary) | `EMIS_LSEC_SNE_LEARNERS_VISION` | Lower Secondary |  |
| SNE learners, vision (Primary) | `EMIS_PRI_SNE_LEARNERS_VISION` | Primary |  |
| SNE learners, vision (Tertiary) | `EMIS_TERT_SNE_LEARNERS_VISION` | Tertiary |  |
| SNE learners, vision (Upper Secondary) | `EMIS_USEC_SNE_LEARNERS_VISION` | Upper Secondary |  |
| Survival rate (Lower Secondary) | `EMIS_LSEC_SURVIVAL_RATE` | Lower Secondary |  |
| Survival rate (Primary) | `EMIS_PRI_SURVIVAL_RATE` | Primary |  |
| Survival rate (Tertiary) | `EMIS_TERT_SURVIVAL_RATE` | Tertiary |  |
| Survival rate (Upper Secondary) | `EMIS_USEC_SURVIVAL_RATE` | Upper Secondary |  |
| Teachers qualified in science (Lower Secondary) | `EMIS_LSEC_TEACHERS_QUALIFIED_SCIENCE` | Lower Secondary |  |
| Teachers qualified in science (Upper Secondary) | `EMIS_USEC_TEACHERS_QUALIFIED_SCIENCE` | Upper Secondary |  |
| Total enrolment as a percentage of the official school-age population (Primary) | `EMIS_PRI_ENROLMENT_PCT_OFFICIAL_AGE_POP` | Primary |  |
| Young people achieving at least a minimum proficiency level in mathematics, end of cycle (Lower Secondary) | `SDG4_LSEC_MPL_MATHS` | Lower Secondary, Household Survey | SDG 4 |
| Young people achieving at least a minimum proficiency level in reading, end of cycle (Lower Secondary) | `SDG4_LSEC_MPL_READING` | Lower Secondary, Household Survey | SDG 4 |
| Young people achieving at least a minimum proficiency level in science, end of cycle (Lower Secondary) | `SDG4_LSEC_MPL_SCIENCE` | Lower Secondary, Household Survey | SDG 4 |
| Young people aged 15-24 achieving at least a minimum proficiency level in mathematics | `SDG4_YOUTH_MPL_MATHS` | Household Survey | SDG 4 |
| Young people aged 15-24 achieving at least a minimum proficiency level in reading | `SDG4_YOUTH_MPL_READING` | Household Survey | SDG 4 |

| Package | Indicators |
|---|---|
| ECD | 61 |
| Primary | 85 |
| Lower Secondary | 72 |
| Upper Secondary | 62 |
| Tertiary | 22 |
| Household Survey | 15 |
| National Expenditure | 7 |
| *distinct indicators in the toolkit* | 245 |
