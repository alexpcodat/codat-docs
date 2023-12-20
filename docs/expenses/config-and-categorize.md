---
title: "Map customer settings"
description: Set up, map, and enable the Sync for Expenses solution
sidebar_label: Map settings
tags: [syncforexpense, mappingOptions, Config]
---

Once your SMB user has authorized a connection to their accounting platform and you have created a data connection, you can then configure and enable Sync for Expenses.

## Create configuration

The [configuration endpoint](/sync-for-expenses-api#/operations/get-company-configuration) enables you to set up how your customers' expenses will be pushed. 
You can check the config anytime to confirm your company's configuration.

```http title="Company Config"
GET https://api.codat.io/companies/{companyId}/sync/expenses/config
{
    "bankAccount": {
        "id": "{selectedBankAccountId}"
    },
    "supplier": {
        "id": "{selectedSupplierId}"
    },
    "customer": {
        "id": "{selectedCustomerId}"
    }
}
```

### Bank Account

- A bank account (`bankAccount.id`) is required to show where purchases have been made from. This can either a credit or debit account. 
- You can either choose to create a new account or retrieve a list of exisiting accounts from your customers accounting software. 
    
    - [GET](/accounting-api#/operations/get-account) a list of available accounts. Use this request to retrieve a list of relevant accounts from your customers' accounting software. You should also add additional query parameters, e.g. `query=metadata.isDeleted=false&&isBankAccount=true`. For **credit cards**, you can use an additional query parameter, e.g. `query=metadata.isDeleted=false&&isBankAccount=true&&type=liability`.
    - To **create** a new bank account you can use the following [POST](/accounting-api#/operations/create-bank-accounts) endpoint. Please note that you should trigger a data refresh [GET](/accounting-api#/operations/bankAccounts) if a new bank account has been created prior to syncing transactions. 

:::info Foreign exchange 💱

There are two ways to handle transactions in a foreign currency:

1. Each foreign exchange currency has its own bank account.

2. Each transaction is converted back to the currency of the bank account.

Sync for Expenses supports both of these options.
:::

### Supplier

- A supplier (`supplier.id`) is required to associate all spending against that supplier. 
    - [GET](/accounting-api#/operations/list-suppliers) a list of available suppliers in the company's accounting software. You can also add additional query parameters, e.g. `query=metadata.isDeleted=false&&supplierName=supplierName`.
    - You can [POST](/accounting-api#/operations/create-suppliers) to create a new supplier.
- The currency associated with the supplier must match the currency associated with the spend. Codat validates the match for suppliers with a single set currency, but not for suppliers that work with multiple currencies.

In some cases, different accounting platforms have certain ways of handling suppliers and customers, based on transaction types: 

<table>
  <thead></thead>
  <tbody>
    <tr>
      <td style={{ textAlign: 'center' }} colspan="6"><b>Supported Platforms</b></td>
    </tr>
    <tr>
      <td></td>
      <td></td>
      <td><b>Xero</b></td>
      <td><b>QBO</b></td>
      <td><b>Netsuite</b></td>
      <td><b>Microsoft Dynamics</b></td>
    </tr>
    <tr>
      <td rowspan="8"><i>Transaction Types</i></td>
      <td>Payments</td>
      <td>Supplier used</td>
      <td>Supplier used</td>
      <td>Supplier used</td>
      <td rowspan="8">Supplier is not associated with expense transactions due to a Dynamics platform limitation.</td>
    </tr>
    <tr>
      <td>Refund</td>
      <td>Customer used</td>
      <td>Supplier used</td>
      <td>Supplier used</td>
    </tr>
    <tr>
      <td>Rewards</td>
      <td>Customer used</td>
      <td>Supplier used</td>
      <td>NA</td>
    </tr>
    <tr>
      <td>Chargeback</td>
      <td>Customer used</td>
      <td>Supplier used</td>
      <td>NA</td>
    </tr>
    <tr>
      <td>Transfer In</td>
      <td>Customer used</td>
      <td>Supplier used</td>
      <td>NA</td>
    </tr>
    <tr>
      <td>Transfer Out</td>
      <td>Supplier used</td>
      <td>Supplier used</td>
      <td>NA</td>
    </tr>
    <tr>
      <td>Adjustment In</td>
      <td>If the chart of accounts’ is a bank account, then supplier used, if not then customer used.</td>
      <td>Customer used</td>
      <td>NA</td>
    </tr>
    <tr>
      <td>Adjustment Out</td>
      <td>Supplier used</td>
      <td>Customer used</td>
      <td>NA</td>
    </tr>
  </tbody>
</table>

### Customer

- Choose which customer (<code>customer.id</code>) any income-related activities, such as cashback, should be associated with.  
  - <a href="/accounting-api#/operations/get-customers">GET</a> A list of available customers, you can add additional query parameters e.g. <code>query=metadata.isDeleted=false&&customerName=name</code>
  - You can <a href="/accounting-api#/operations/post-customers">POST</a> to the accounting API to create a new customer.

You can then [post](sync-for-expenses-api#/operations/set-company-configuration) the updated configuration to Codat to store their saved preferences.

### Overriding the configuration settings

- Suppliers and Bank Accounts can be set at the configuration level and at the [transaction level](https://docs.codat.io/sync-for-expenses-api#/operations/create-expense-dataset#request-body).
    - By setting these at the transaction level, you can override the configuration and sync a more accurate representation of who or where the spend should be associated with in the accounting platform. 
    - If no override is set at the transaction level, the spend will have the configured supplier or bank account set as a default against it. 
    - This functionality is not currently supported for Customers.

``` http title="Bank Account override on the expense transaction"
      "bankAccountRef":{
          "id":"08ca1f02-0374-11ed-b939-0242ac120002",
```
``` http title="Supplier override on the expense transaction"
     "contactRef":{
          "id":"08ca1f02-0374-11ed-b939-0242ac120002",
          "type": "Supplier"
```  

## Mapping options

Every company has its own preference on how an individual expense can be represented in their accounting software. You can retrieve these options from the `mappingOptions` [endpoint](/sync-for-expenses-api#/operations/get-mapping-options).

The `mappingOptions` response can then be cached and displayed to the end user when they are finalizing their expenses before exporting.

```json title="Sample mappingOptions response"
{
    "expenseProvider": "Partner Expense",
    "accounts": [
        {
            "id": "c5194f9d-b443-4630-b2d4-339bd57d313c",
            "name": "Interest Earned",
            "currency": "GBP",
            "accountType": "Asset",
            "validTransactionTypes": [
                "Reward",
                "Adjustment",
                "Transfer"
            ]
        }
        ...
    ],
    "trackingCategories": [
        {
            "id": "dba3d4da-f9ed-4eee-8e0b-452d11fdb1fa",
            "modifiedDate": "2022-08-03T12:04:40.067Z",
            "name": "Sales and Marketing",
            "hasChildren": false,
            "parentId": "DEPARTMENT"
        }
        ...
    ],
    "taxRates": [
        {
            "id": "INPUT2",
            "name": "20% (VAT on Expenses)",
            "code": "INPUT2",
            "effectiveTaxRate": 20,
            "totalTaxRate": 20
        }
        ...
    ]
}
```

You will normally see the name of the connected provider in the `expenseProvider` property.

**Accounts**

The `accounts` array will include the general ledger accounts which have been pulled from the business's accounting software. The `name` is what they have labeled the account in their accounting software, so you can display this to your end user.

`validTransactionTypes` will tell you which types are accepted by the account. This prevents validation issues such as an SMB accidentally trying to reconcile an expense to an income account.

Additional accounts can also be created using the public API, this could be useful if a company has a new category for representing expenses.

``` http title="create new expense account"
POST https://api.codat.io/companies/{companyId}/connections/{connectionId}/push/accounts",
{
    "nominalCode": "310",
    "name": "Stationary Costs",
    "fullyQualifiedCategory": "Expenses"
}
```

**Tracking categories**

Tracking categories are used to monitor cost centers and control budgets that sit outside of the standard chart of accounts. Your customers may use tracking categories to group and track the income and costs of specific departments, such as "sales and marketing", projects, locations, or customers.

When pushing an expense reconciliation, you can include a tracking category to associate this to further categorize an expense.

**Tax rates**

The tax rates enable your SMBs to accurately track taxes against purchases and, depending on the locale, allow them to recoup the tax.

Accounting systems typically store a set of taxes and associated rates within the accounting package. This means users don't have to look up or remember the rate for each type of tax. For example, applying the tax "UK sales VAT" to the line items of an invoice adds the correct tax rate of 20%. 

Assigning a tax rate to a transaction is mandatory unless the transaction is a transferIn or transferOut (see [Transaction types](/expenses/sync-process/expense-transactions#transaction-types). In some cases, your customers might not need to track tax on expenses. We recommend assigning a default tax code for 0% from the accounting package for those transactions.

**Refreshing the mappingOptions**

The default [sync settings](/expenses/getting-started#datatypes) will refresh the mapping options on an daily basis, however, you can also refresh the options by making the following request.

``` http
POST https://api.codat.io/companies/{companyId}/data/all
```
