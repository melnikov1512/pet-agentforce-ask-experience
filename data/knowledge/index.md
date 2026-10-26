# Reusable Salesforce Knowledge Articles

`all-articles.json` contains all 100 English Salesforce basic-setup articles in one reusable dataset. It is a JSON array of creation-field objects for `Knowledge__kav`, with HTML content in `Article_Body__c`.

Each record includes `Title`, `UrlName`, `Summary`, `Language`, `Article_Body__c`, and public, customer, and partner visibility flags set to `false`. Source-org IDs, article numbers, and publication status are excluded.

## Create articles in another org

1. Confirm Knowledge is enabled and the importing user has the required authoring and publication access.
2. Deploy the repository's `Knowledge__kav.Article_Body__c` field and appropriate field permissions if missing. Validate field lengths, enabled language, record types, and any additional required fields in the destination org.
3. Check existing articles by URL name and language before inserting; this file is not an automatic upsert file.
4. Adapt the array to the selected supported API or import tool. It is not a Salesforce CLI tree-import plan or Bulk API CSV. For an sObject Collections request, add the required request envelope and `Knowledge__kav` type attributes.
5. Create drafts, retain returned IDs and per-record results, and reconcile any partial success before retrying.
6. Publish the intended articles through the supported Knowledge publication lifecycle. Metadata deployment alone does not create or publish article records.
7. Verify the destination has all 100 intended published articles, matching content, and the intended visibility.

The setup procedures in the content require review against the destination org's edition, licenses, enabled features, and release. Future Agentforce ingestion and access configuration are separate steps; include `Article_Body__c` in the chosen knowledge-source mapping and verify retrieval.
