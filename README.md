FlindersDSpace-OrcidDisplay
===========================

## Summary

Workaround to display the ORCID® of any author (including the Chief
Investigator A) in DSpace. 

## Background

The Australian NHMRC have specified the following in the document *National
Health and Medical Research Council, Open Access Policy, 15 January 2018,
Appendix 1, item 10*:

> "The [institutional repository] publication metadata must also include
> the ORCID identifier of the author submitting the metadata."

The NHMRC have also confirmed on the CAIRSS list that they:

> "... require the ORCID of the CIA to be displayed"

The Australian Research Council have specified something similar in
the document *ARC Open Access Policy Version 2017.1, section 6.3.2*:

> Metadata should also contain other relevant information as
> applicable and appropriate to the Research Output including,
> but not limited to: ... the ORCID identifier for the author
> responsible for providing the Research Output (to be made
> Openly Accessible).

However, despite the fact that DSpace 5 and 6 include ORCID authority
control integration, that integration does *not* display the ORCID hence
cannot comply with the above statements. (There have also been some
issues regarding the decommissioning of ORCID API v1 and an apparent
ORCID authority control bug in DSpace 5.9 and 6.3, but that is a
separate story.)

So, is there a way to integrate the ORCID of the Chief Investigator A
into DSpace?

Desirable features:
1. Easy to setup.
2. Easy to maintain.
3. Easy to decommission later (eg. when DSpace supports the display of ORCIDs).
4. Doesn't pollute or degrade metadata collected by harvesters and web
   crawlers.

References:
- https://www.nhmrc.gov.au/grants-funding/policy/nhmrc-open-access-policy
- https://www.nhmrc.gov.au/_files_nhmrc/file/research/nhmrc_open_access_policy_15_january_2018_v2.pdf
- https://www.arc.gov.au/policies-strategies/policy/arc-open-access-policy-version-20171
- https://groups.google.com/forum/#!topic/dspace-tech/HO_4e-WYjjo
- https://jira.duraspace.org/browse/DS-3447
- https://support.orcid.org/hc/en-us/articles/360006897674-Structure-of-the-ORCID-Identifier
- https://groups.google.com/forum/#!topic/dspace-tech/4nJRVyn8Hk8


## The solution

### Environment
- DSpace 5.9
- XMLUI Mirage 2

### Setup tasks
- In the web user interface, create a new schema called "local". Eg.
  * Login as an admin 
  * Navigate to Registries: Metadata > Add a new schema
  * Namespace: http://localhost.localdomain/metadata/schema/local
  * Name: local
  * Click "Add a new schema"
- Create a new field within the "local" schema called
  "local.contributor.authorOrcidLookup". Eg. 
  * Login as an admin
  * Navigate to Registries: Metadata > Name: local
  * First "Field Name" text box: contributor
  * Second "Field Name" text box: authorOrcidLookup
  * Scope Note: A field to hold "AUTHOR: ORCID-URL" where the AUTHOR
    subfield is a lookup key which matches dc.contributor.author
    (or dc.creator or dc.contributor)
  * Click "Add a new metadata field"
- Apply any of the file changes within this Git repository which are
applicable to your DSpace 5.9 XMLUI Mirage 2 site.

### How it works
- <sup>[1](#fnote1)</sup>
  Enter the (ordered) list of authors as usual.

- <sup>[1](#fnote1)</sup>
  For each author to be associated with an ORCID (e.g. Chief Investigators
  and/or authors at your institution) enter a separate field
  (local.contributor.authorOrcidLookup) containing the ORCID-URL with
  author as the lookup key in the format "AUTHOR: ORCID-URL" (where colon
  ":" is the delimiter between the author (key) and the ORCID-URL). E.g.
  "Smith, Donald Jr: https://orcid.org/0000-1111-2222-333X". Dublin Core
  schema fields (ie. dc.xxx.yyy) appear directly in item page META-tags
  and OAI-PMH harvests. Because we do not want this behaviour for our
  new field (we only want the ORCID-URL component from this field to
  appear *indirectly*) the new field was created within a different
  schema (i.e. "local").

- <sup>[2](#fnote2)</sup>
  In the data entry submission form, add the
  local.contributor.authorOrcidLookup entry-field directly beneath
  the author entry-field so it is easy to copy-and-paste author names
  from one field to the other (since they must be identical for the
  lookup-by-author key to behave correctly).

- <sup>[3](#fnote3)</sup> <sup>[4](#fnote4)</sup>
  On the Simple Item Record page, against any author which has an
  associated local.contributor.authorOrcidLookup field populated, add
  an ORCID icon with the associated ORCID-URL as a hyperlink. Note that
  the author list will be ordered the same as DSpace 5.9 normal behaviour
  and is not influenced by the order of the
  local.contributor.authorOrcidLookup fields. Note that it is ok for
  zero, one, a few or all authors to have an
  associated local.contributor.authorOrcidLookup field.

- <sup>[5](#fnote5)</sup>
  On the Simple and Full Item Record pages, add a META-tag for
  DC.relation with the content of the ORCID-URL for each
  local.contributor.authorOrcidLookup field.

- <sup>[6](#fnote6)</sup>
  On OAI-PMH records for the "oai_dc" metadata format, add a
  &lt;dc:relation&gt; element with the content of the ORCID-URL for
  each local.contributor.authorOrcidLookup field. This was the field
  and format recommended by [Trove](https://trove.nla.gov.au/)
  (as our institutional repository is harvested into Trove).

### Notes
<a name="fnote1">1</a>: Performed by a DSpace user.

<a name="fnote2">2</a>: Performed by a program within this Git repository.
See [config/input-forms.xml](config/input-forms.xml)

<a name="fnote3">3</a>: Performed by a program within this Git repository.
See [webapps/xmlui/themes/Mirage2/xsl/aspect/artifactbrowser/item-view.xsl](webapps/xmlui/themes/Mirage2/xsl/aspect/artifactbrowser/item-view.xsl);
[webapps/xmlui/themes/Mirage2/images/orcid.png](webapps/xmlui/themes/Mirage2/images/orcid.png)

<a name="fnote4">4</a>: The ORCID 16x16 PNG icon was downloaded from
[here](https://orcid.org/trademark-and-id-display-guidelines).

<a name="fnote5">5</a>: Performed by a program within this Git repository.
See [webapps/xmlui/themes/Mirage2/xsl/core/page-structure.xsl](webapps/xmlui/themes/Mirage2/xsl/core/page-structure.xsl)

<a name="fnote6">6</a>: Performed by a program within this Git repository.
See [config/crosswalks/oai/metadataFormats/oai_dc.xsl](config/crosswalks/oai/metadataFormats/oai_dc.xsl)

7: See the [screenshots](dspaceOrcidDisplay.pdf)

### Benefits
- Complies with NHMRC/ARC requirements above.
- On the Simple Item Record page, the addition of ORCID info does not use
  much web-page realestate.
- The ORCID and hyperlink is easily visible via the human user interface.
- The ORCID hyperlink is available to web crawlers via the DC.relation META-tag.
- The ORCID hyperlink is available to OAI-PMH harvest clients/consumers
  (such as Trove).
- The ORCID-URL is only entered once (i.e. in the
  local.contributor.authorOrcidLookup field).
- The messy local.contributor.authorOrcidLookup field does not appear
  *directly* in the Simple Item Record page, or any META-tags or any
  OAI-PMH &lt;dc:relation&gt; elements. (As expected, it does appear
  in the Full Item Record page in the list of metadata fields.)

### Gotchas
- Author names must match precisely in terms of case-sensitivity and the
  position of characters including commas, leading/trailing space etc.
- The order of DC.relation META-tags will be the same as the order of the
  local.contributor.authorOrcidLookup fields.
- The order of &lt;dc:relation&gt; OAI-PMH XML elements will be the same as the
  order of the local.contributor.authorOrcidLookup fields.
- Although these DC.relation META-tags and &lt;dc:relation&gt; OAI-PMH XML
  elements (derived from the local.contributor.authorOrcidLookup field)
  are available to machines (i.e. web crawlers and harvest clients) they
  do *not* exist within the DSpace database and hence do not exist within the
  [DSpace Intermediate Metadata](https://wiki.duraspace.org/display/DSPACE/DSpaceIntermediateMetadata)

### OAI-PMH test procedure guideline
1. Update oai_dc.xsl
2. Restart tomcat (or other Java web server/Java Servlet Container running
   DSpace). E.g. takes 1-3 minutes
3. Update your test record (in DSpace 5.9 XMLUI Mirage2)
4. [dspace]/bin/dspace oai clean-cache  # Optional? E.g. takes 6 seconds
5. [dspace]/bin/dspace oai import -o > /dev/null  # E.g. takes 12 seconds (but took 35 minutes first time)
6. View the OAI-PMH changes, e.g.
   https://dspace.example.com/oai/request?verb=ListRecords&metadataPrefix=oai_dc&set=com_123456789_36019

## Related repositories
- This Github repository supersedes [grantj-re3/FlindersDSpace-OrcidCIA](https://github.com/grantj-re3/FlindersDSpace-OrcidCIA).
- [The DSpace digital asset management system](https://github.com/DSpace/DSpace)

