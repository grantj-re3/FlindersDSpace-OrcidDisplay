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
- https://groups.google.com/forum/#!topic/dspace-tech/HO_4e-WYjjo
- https://jira.duraspace.org/browse/DS-3447
- https://support.orcid.org/knowledgebase/articles/116780-structure-of-the-orcid-identifier
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
- Enter the (ordered) list of authors as usual.
- For each author to be associated with an ORCID (e.g. Chief Investigators
  and/or authors at your institution) enter a separate field
  (local.contributor.authorOrcidLookup) containing the ORCID-URL with
  author-name as the lookup key, eg.
  "Smith, Donald Jr: https://orcid.org/0000-1111-2222-333X". Dublin Core
  schema fields (ie. dc.xxx.yyy) appear directly in item page META-tags
  and OAI-PMH harvests. Because we do not want this behaviour for our
  new field (we only want the ORCID-URL component from this field to
  appear *indirectly*) the new field was created within a different
  schema (i.e. "local").
- [%] In the data entry submission form, add the
  local.contributor.authorOrcidLookup entry-field directly beneath
  the author entry-field so it is easy to copy-and-paste author names
  from one field to the other (since they must be identical for the
  lookup-by-author key to behave correctly). [See config/input-forms.xml]
- [%] On the Simple Item Record page, against any author which has an
  associated local.contributor.authorOrcidLookup field populated, add
  an ORCID icon with the associated ORCID-URL as a hyperlink. Note that
  the author list will be ordered the same as DSpace 5.9 normal behaviour
  and is not influenced by the order of the
  local.contributor.authorOrcidLookup fields. Note that it is ok for
  zero, one, a few or all authors to have an
  associated local.contributor.authorOrcidLookup field. [See
  webapps/xmlui/themes/Mirage2/xsl/aspect/artifactbrowser/item-view.xsl;
  webapps/xmlui/themes/Mirage2/images/orcid.png]
- [%] On the Simple and Full Item Record pages, add a META-tag for
  DC.relation with the content of the ORCID-URL for each
  local.contributor.authorOrcidLookup field. [See
  webapps/xmlui/themes/Mirage2/xsl/core/page-structure.xsl]
- [%] On OAI-PMH records for the "oai_dc" metadata format, add a
  <dc:relation> field with the content of the ORCID-URL for each
  local.contributor.authorOrcidLookup field. This was the field
  and format recommended by [Trove](https://trove.nla.gov.au/)
  (as our institutional repository is harvested into Trove).
  [See config/crosswalks/oai/metadataFormats/oai_dc.xsl]

Notes:

[%] = Done automatically via a program within this Git repository.

See screenshot...

### Benefits
- Complies with NHMRC requirements above.
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
  OAI-PMH <dc:relation> fields. (As expected, it does appear in the
  Full Item Record page in the list of metadata fields.)

### Gotchas
- Author names must match precisely in terms of case-sensitivity and the
  position of characters including spaces, commas, etc.
- The order of DC.relation META-tags will be the same as the order of the
  local.contributor.authorOrcidLookup fields.
- The order of <dc:relation> OAI-PMH XML fields will be the same as the
  order of the local.contributor.authorOrcidLookup fields.
- Although these DC.relation META-tags and <dc:relation> OAI-PMH XML
  fields (derived from the local.contributor.authorOrcidLookup field)
  are available to machines (i.e. web crawlers and harvest clients) they
  do *not* exist within the DSpace database and hence do not exist within the
  [DSpace Intermediate Metadata](https://wiki.duraspace.org/display/DSPACE/DSpaceIntermediateMetadata)

### OAI-PMH test procedure guideline
1. Update oai_dc.xsl
2. Restart tomcat (or other Java web server/Java Servlet Container running
   DSpace). E.g. took 1-3 minutes
3. Update your test record (in DSpace 5.9 XMLUI Mirage2)
4. [dspace]/bin/dspace oai clean-cache  # Optional? E.g. took 6 seconds
5. [dspace]/bin/dspace oai import -o > /dev/null  # E.g. took 12 seconds (but 35 minutes first time)
6. View the OAI-PMH changes, e.g.
   https://dspace.example.com/oai/request?verb=ListRecords&metadataPrefix=oai_dc&set=com_123456789_36019

