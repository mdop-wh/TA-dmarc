# TA-dmarc add-on for Splunk

Add-on for ingesting DMARC aggregate reports into Splunk

## Install the TA-dmarc add-on for Splunk

### Single instance Splunk deployments

1. In Splunk, click on "Manage Apps"
2. Click "Browse more apps", search for "TA-dmarc" and install the add-on

### Distributed Splunk deployments

| Instance type | Supported | Required | Description
|---------------|-----------|----------|------------
| Search head   | Yes       | Yes      | Install this add-on on your search head(s) where CIM compliance of DMARX aggregate reports is required
| Indexer       | Yes       | No       | This add-on should be installed on a heavy forwarder that does the index time parsing. There is no need to install this add-on on an indexer too.
| Universal Forwarder | No  | No       | This add-on is not supported on a Universal Forwarder because it requires Python
| Heavy Forwarder     | Yes | Yes      | Install this add-on on a heavy forwarder to ingest DMARC XML aggregate reports into Splunk.

The following table lists support for distributed deployment roles in a Splunk deployment:

| Deployment role | Supported | Description
|-----------------|-----------|-------------
| Search head deployer | Yes  | Install this add-on on your search head deployer to enable CIM compliance of DMARC aggregate reports on a Search Head Cluster
| Cluster Master       | No  | This add-on should be installed on a heavy forwarder that performs parsing at index time. There is no need to install this add-on on an indexer too.
| Deployment Server    | Yes  | Install this add-on on your Deployment Server to deploy it to search heads and heavy forwarders.

## Configure inputs for TA-dmarc

1. Go to the add-on's configuration UI and configure a new modular input by clicking on the "Inputs" menu.
2. Click create new input
3. Configure:
   * Name: e.g. "dmarc-inbox"
   * Interval: how often to poll the directory where DMARC XML aggregate reports are dropped (see below)
   * Index: what Splunk index to send the aggregate reports to
   * Directory: Location where DMARC aggregate report files are dropped
   * Quiet time: Ignore files that have a modification time of less than n seconds ago. You can use this to prevent ingesting large files that are dropped on a share but take some time to transfer
   * Resolve IP: Whether to resolve the row source_ip in the DMARC XML aggregate reports
4. Click add

## DMARC aggregate reports

This add-on handles the following file formats in which aggregate reports are delivered:

1. XML (as .xml file)
2. ZIP (as .zip)
3. GZ (as .xml.gz)

Mitigations are in place against:

* ZIP bombs
* gzip bombs
* various XML attack vectors like billion laughs, quadratic blowup, external entity expansion and so on thanks to the defusedxml library

### Field mapping

From the XML sample below, these fields are created:

| XML field                       | Splunk field name               | Value                                       |
|---------------------------------|---------------------------------|---------------------------------------------|
|feedback/report_metadata/org_name | rpt_metadata_org_name            | google.com                                  | 
|feedback/report_metadata/email    | rpt_metadata_email               | noreply-dmarc-support@google.com            | 
|feedback/report_metadata/extra_contact_info | rpt_metadata_extra_contact_info  | https://support.google.com/a/answer/2466580 | 
|feedback/report_metadata/report_id | rpt_metadata_report_id           | 13190401177475355109                        | 
|feedback/report_metadata/date_range/begin | rpt_metadata_date_range_begin    | 1506988800                                  | 
|feedback/report_metadata/date_range/end | rpt_metadata_date_range_end      | 1507075199                                  | 
|feedback/policy_published/domain  | policy_published_domain          | myfirstdomain.tld                           | 
|feedback/policy_published/adkim   | policy_published_adkim           | r                                           | 
|feedback/policy_published/adpf    | policy_published_aspf            | r                                           | 
|feedback/policy_published/p       | policy_published_p               | none                                        | 
|feedback/policy_published/pct     | policy_published_pct             | 100                                         | 
|feedback/record/row/source_ip     | row_source_ip                    | 256.32.191.194                              | 
|feedback/record/row/count         | row_count                        | 1                                           | 
|feedback/record/row/policy_evaluated/disposition |row_policy_evaluated_disposition | none                                        | 
|feedback/record/row/policy_evaluated/dkim |row_policy_evaluated_dkim        | fail                                        | 
|feedback/record/row/policy_evaluated/spf  |row_policy_evaluated_spf         | fail                                        | 
|feedback/record/identifiers/header_from   |identifiers_header_from          | myfirstdomain.tld                           | 
|feedback/record/auth_results/spf/domain   | auth_result_spf_domain           | myfirstdomain.tld                           | 
|feedback/record/auth_results/spf/domain   | auth_result_spf_result           | fail                                        | 

### Authentication datamodel

Besides the fields contained in the report, additional fields are mapped from the CIM Authentication datamodel, based on the XML sample below:

| Authentication datamodel field name  | Value                           |
|--------------------------------------|---------------------------------|
| action       | failure               |
| app          | dmarc                 |
| dest         | google.com            |
| signature    | Use of mail-from domain myfirstdomain.tld |
| signature_id | 13190401177475355109  |
| src          | 256.32.191.194 |
| src_ip       | 256.32.191.194 |
| eventtype    | dmarc_rua_spf_only |
| tag          | authentication, insecure|


### DMARC XML sample

```
<?xml version="1.0" encoding="UTF-8" ?>
<feedback>
  <report_metadata>
    <org_name>google.com</org_name>
    <email>noreply-dmarc-support@google.com</email>
    <extra_contact_info>https://support.google.com/a/answer/2466580</extra_contact_info>
    <report_id>13190401177475355109</report_id>
    <date_range>
      <begin>1506988800</begin>
      <end>1507075199</end>
    </date_range>
  </report_metadata>
  <policy_published>
    <domain>myfirstdomain.tld</domain>
    <adkim>r</adkim>
    <aspf>r</aspf>
    <p>none</p>
    <sp>none</sp>
    <pct>100</pct>
  </policy_published>
  <record>
    <row>
      <source_ip>256.32.191.194</source_ip>
      <count>1</count>
      <policy_evaluated>
        <disposition>none</disposition>
        <dkim>fail</dkim>
        <spf>fail</spf>
      </policy_evaluated>
    </row>
    <identifiers>
      <header_from>myfirstdomain.tld</header_from>
    </identifiers>
    <auth_results>
      <spf>
        <domain>myfirstdomain.tld</domain>
        <result>fail</result>
      </spf>
    </auth_results>
  </record>
</feedback>
```


## Third party software credits

The following software components are used in this add-on:

1. [defusedxml](https://pypi.python.org/pypi/defusedxml/0.5.0) by Christian Heimes
2. [Splunk Add-on Builder](https://docs.splunk.com/Documentation/AddonBuilder/2.2.0/UserGuide/Overview) by Splunk and the [third-party software](https://docs.splunk.com/Documentation/AddonBuilder/2.2.0/UserGuide/Thirdpartysoftwarecredits) it uses
