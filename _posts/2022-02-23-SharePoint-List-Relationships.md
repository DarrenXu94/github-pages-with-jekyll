---
title: "SharePoint List Relationships"
date: 2022-02-23
---

I have been using SharePoint for a while and recently discovered the lookup column linking capability. This seemed like an extremely handy feature but I didn't even realise I was already using it when I added the User column types to my list items. Essentially there is a column in SharePoint that you can specify as a key to another list.

**TLDR: Like a poor man's GraphQL with limited column type support and only one level of nesting**

    <Field ID="{GUID}" Name="Lookup" StaticName="Lookup" DisplayName="Lookup" Type="Lookup" List="{Lookup List GUID}" ShowField="[Lookup Internal Field Name]" />

I added in the following column to my list (Users) and linked it to another list (Jobs).

    <Field Group="Sample" ID="{7f51d69e-9cf7-4158-bbd7-0e1347ac73d3}" Name="JobLookup" StaticName="JobLookup" DisplayName="JobLookup" Type="Lookup" List="{14e1c914-bde7-4ffe-ab24-b0babf25c4b8}" ShowField="Title" Mult="TRUE" />

To access a 'deep' key of an object you need to use the expand command in pnpjs.

Firstly I tested the call without expand to see what it would return.

    return await sp.web.lists.getByTitle(list).items.get();

    // returns JobLookupId: [3]

Next I tried querying with some default columns.

    return await sp.web.lists
          .getByTitle(list)
          .items.select(
            "Title",
            "JobLookup/Title",
            "JobLookup/Modified"
          )
          .expand("JobLookup");

    /* returns JobLookup: Array(1)
    0:
    Modified: "2021-07-28T07:24:45Z"
    Title: "Job info"
    odata.id: "f2d2b5b4-547b-49f2-9ffd-c695d578feef"
    odata.type: "SP.Data.JobsListItem" */

Finally I tried querying with my own custom field. This is where it fell over.

    return await sp.web.lists
          .getByTitle(list)
          .items.select(
            "Title",
            "JobLookup/Title",
            "JobLookup/Modified",
            "JobLookup/JobInfo"
          )
          .expand("JobLookup");

    // "message":{"lang":"en-US","value":"The query to field 'JobLookup/JobInfo' is not valid."}      

According to official [Microsoft documentation](https://support.microsoft.com/en-us/office/create-list-relationships-by-using-unique-and-lookup-columns-80a3e0a6-8016-41fb-ad09-8bf16d490632?ui=en-us&rs=en-us&ad=us), there are only a few column types that can be expanded. As well as that "You can't index a secondary column or make a secondary column unique."

<table id="tblID0EBBFAAA" class="banded flipColors" style="box-sizing: border-box; border-collapse: collapse; margin-bottom: 20px; border-bottom: 1px solid rgb(204, 204, 204); width: 673.188px; margin-top: 10px; padding: 0px; line-height: normal; border-top: 1px solid rgb(204, 204, 204); color: rgb(54, 54, 54); font-family: &quot;Segoe UI&quot;, &quot;Segoe UI Web Regular&quot;, &quot;Segoe UI Symbol&quot;, &quot;Helvetica Neue&quot;, &quot;BBAlpha Sans&quot;, &quot;S60 Sans&quot;, Arial, sans-serif; font-size: 10px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; background-color: rgb(255, 255, 255); text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;">

<thead style="box-sizing: border-box;">

<tr style="box-sizing: border-box; vertical-align: top; padding: 0px !important; background-color: rgb(218, 218, 218); border-top: 1px solid rgb(204, 204, 204); border-bottom: 1px solid rgb(204, 204, 204); text-transform: none;">

<th class="" style="box-sizing: border-box; text-align: left; padding: 3px 10px 3px 5px; background: rgb(218, 218, 218); border: 0px; font-size: 1em; color: rgb(51, 51, 51); text-transform: none; font-weight: normal;">

**Supported Column Types**

</th>

<th style="box-sizing: border-box; text-align: left; padding: 3px 10px 3px 5px; background: rgb(218, 218, 218); border: 0px; font-size: 1em; color: rgb(51, 51, 51); text-transform: none; font-weight: normal;">

**Unsupported Column Types**

</th>

</tr>

</thead>

<tbody style="box-sizing: border-box;">

<tr style="box-sizing: border-box; vertical-align: top; padding: 0px !important; background-color: rgb(244, 244, 244); border-top: 1px solid rgb(204, 204, 204); border-bottom: 1px solid rgb(204, 204, 204);">

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;">

Single line of text

</td>

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;">

Multiple lines of text

</td>

</tr>

<tr style="box-sizing: border-box; vertical-align: top; padding: 0px !important; background-color: rgb(244, 244, 244); border-top: 1px solid rgb(204, 204, 204); border-bottom: 1px solid rgb(204, 204, 204);">

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;">

Number

</td>

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;">

Currency

</td>

</tr>

<tr style="box-sizing: border-box; vertical-align: top; padding: 0px !important; background-color: rgb(244, 244, 244); border-top: 1px solid rgb(204, 204, 204); border-bottom: 1px solid rgb(204, 204, 204);">

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;">

Date and Time

</td>

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;">

Person or Group

</td>

</tr>

<tr style="box-sizing: border-box; vertical-align: top; padding: 0px !important; background-color: rgb(244, 244, 244); border-top: 1px solid rgb(204, 204, 204); border-bottom: 1px solid rgb(204, 204, 204);">

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;"></td>

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;">

Calculated

</td>

</tr>

<tr style="box-sizing: border-box; vertical-align: top; padding: 0px !important; background-color: rgb(244, 244, 244); border-top: 1px solid rgb(204, 204, 204); border-bottom: 1px solid rgb(204, 204, 204);">

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;"></td>

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;">

Hyperlink or Picture

</td>

</tr>

<tr style="box-sizing: border-box; vertical-align: top; padding: 0px !important; background-color: rgb(244, 244, 244); border-top: 1px solid rgb(204, 204, 204); border-bottom: 1px solid rgb(204, 204, 204);">

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;"></td>

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;">

Custom Columns

</td>

</tr>

<tr style="box-sizing: border-box; vertical-align: top; padding: 0px !important; background-color: rgb(244, 244, 244); border-top: 1px solid rgb(204, 204, 204); border-bottom: 1px solid rgb(204, 204, 204);">

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;"></td>

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;">

Yes/No

</td>

</tr>

<tr style="box-sizing: border-box; vertical-align: top; padding: 0px !important; background-color: rgb(244, 244, 244); border-top: 1px solid rgb(204, 204, 204); border-bottom: 1px solid rgb(204, 204, 204);">

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;"></td>

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;">

Choice

</td>

</tr>

<tr style="box-sizing: border-box; vertical-align: top; padding: 0px !important; background-color: rgb(244, 244, 244); border-top: 1px solid rgb(204, 204, 204); border-bottom: 1px solid rgb(204, 204, 204);">

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;"></td>

<td style="box-sizing: border-box; padding: 4px 10px; margin: 0px; vertical-align: top;">

Lookup

</td>

</tr>

</tbody>

</table>

This makes this feature limited but still useful in cases such as querying for a user's name but the lack of nesting objects and column types is still a bit disappointing.
