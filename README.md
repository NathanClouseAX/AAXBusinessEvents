# AAXBusinessEvents

Custom business events package for Microsoft Dynamics 365 Finance and Operations, published by [Atomic AX](https://www.atomicax.com).

This package extends the standard D365FO business events framework with process-level events that are not available out of the box. Each event follows the standard `BusinessEventsBase` / `BusinessEventsContract` pattern and can be configured through **System Administration > Business Events**.

## Business Events

### Vendor

| Event | Class | Trigger | Payload |
|-------|-------|---------|---------|
| Vendor Created | `AAXVendorCreatedBusinessEvent` | `VendTable.insert()` | legalEntity, vendorAccount, vendorName |
| Vendor Updated | `AAXVendorUpdatedBusinessEvent` | `VendTable.update()` | legalEntity, vendorAccount, vendorName |

### Production

| Event | Class | Trigger | Payload |
|-------|-------|---------|---------|
| Production Order Scheduled | `AAXProdOrderScheduledBusinessEvent` | `ProdTable.update()` &rarr; `ProdStatus::Scheduled` | legalEntity, productionOrderId, itemId, itemName, productionStatus |
| Production Order Started | `AAXProdOrderStartedBusinessEvent` | `ProdTable.update()` &rarr; `ProdStatus::StartedUp` | legalEntity, productionOrderId, itemId, itemName, productionStatus |
| Production Order Estimated | `AAXProdOrderEstimatedBusinessEvent` | `ProdUpdEstimation.run()` | legalEntity, productionOrderId, itemId |
| Pick List Posted | `AAXProdPickListPostedBusinessEvent` | `ProdJournalTable.update()` &rarr; `Posted` & `PickingList` | legalEntity, productionOrderId, itemId, itemName, productionStatus, goodQuantity, errorQuantity |
| Route Card Posted | `AAXProdRouteCardPostedBusinessEvent` | `ProdJournalTable.update()` &rarr; `Posted` & `RouteCard` | legalEntity, productionOrderId, itemId, itemName, productionStatus, goodQuantity, errorQuantity |
| Report as Finished | `AAXProdReportFinishedBusinessEvent` | `ProdTable.update()` &rarr; `ProdStatus::ReportedFinished` | legalEntity, productionOrderId, itemId, itemName, productionStatus, goodQuantity, errorQuantity |
| Production Order Ended | `AAXProdOrderEndedBusinessEvent` | `ProdTable.update()` &rarr; `ProdStatus::Completed` | legalEntity, productionOrderId, itemId, itemName, productionStatus, goodQuantity, errorQuantity |
| BOM Calculation Completed | `AAXBOMCalcCompletedBusinessEvent` | `BOMCalcTable.insert()` | legalEntity, itemId, costPrice |

### Sales

| Event | Class | Trigger | Payload |
|-------|-------|---------|---------|
| Sales Order Confirmed | `AAXSalesOrderConfirmedBusinessEvent` | `CustConfirmJour.insert()` | legalEntity, salesOrderId, customerAccount, confirmId |
| Sales Packing Slip Posted | `AAXSalesPackingSlipPostedBusinessEvent` | `CustPackingSlipJour.insert()` | legalEntity, salesOrderId, customerAccount, packingSlipId, deliveryDate |
| Sales Return Order Created | `AAXSalesReturnOrderCreatedBusinessEvent` | `SalesTable.insert()` &rarr; `SalesType::ReturnItem` | legalEntity, returnOrderId, customerAccount, returnReasonCodeId |

### Purchasing

| Event | Class | Trigger | Payload |
|-------|-------|---------|---------|
| Purchase Return Order Created | `AAXPurchReturnOrderCreatedBusinessEvent` | `PurchTable.insert()` &rarr; `PurchaseType::ReturnItem` | legalEntity, returnOrderId, vendorAccount, returnReasonCodeId |

### Inventory

| Event | Class | Trigger | Payload |
|-------|-------|---------|---------|
| Transfer Order Shipped | `AAXTransferOrderShippedBusinessEvent` | `InventTransferTable.update()` &rarr; `InventTransferStatus::Shipped` | legalEntity, transferOrderId, fromWarehouse, toWarehouse, transferStatus |
| Transfer Order Received | `AAXTransferOrderReceivedBusinessEvent` | `InventTransferTable.update()` &rarr; `InventTransferStatus::Received` | legalEntity, transferOrderId, fromWarehouse, toWarehouse, transferStatus |
| Inventory Closing Completed | `AAXInventClosingCompletedBusinessEvent` | `InventCostClosing.run()` | legalEntity, closingDate |

### Finance

| Event | Class | Trigger | Payload |
|-------|-------|---------|---------|
| General Journal Posted | `AAXGeneralJournalPostedBusinessEvent` | `LedgerJournalTable.update()` &rarr; `Posted` & `LedgerJournalType::Daily` | legalEntity, journalNumber, journalName, description |
| Free Text Invoice Posted | `AAXFreeTextInvoicePostedBusinessEvent` | `CustInvoiceJour.insert()` (no SalesId) | legalEntity, invoiceId, customerAccount, invoiceDate, invoiceAmount, currencyCode |
| Fixed Asset Acquisition Posted | `AAXFixedAssetAcquisitionPostedBusinessEvent` | `AssetTrans.insert()` &rarr; `AssetTransType::Acquisition` | legalEntity, assetId, bookId, acquisitionAmount, transDate |

### Planning

| Event | Class | Trigger | Payload |
|-------|-------|---------|---------|
| MRP Run Completed | `AAXMRPRunCompletedBusinessEvent` | `ReqCalc.run()` | legalEntity, planId |

### Budget

| Event | Class | Trigger | Payload |
|-------|-------|---------|---------|
| Budget Check Failed | `AAXBudgetCheckFailedBusinessEvent` | `BudgetSource.checkBudget()` (result != Passed) | legalEntity, sourceDocumentType, transactionAmount |

## Architecture

Each business event consists of three components:

### 1. Event Class (extends `BusinessEventsBase`)

Decorated with the `[BusinessEvents]` attribute specifying the contract class, label IDs, and module. Contains:
- A private source field (table buffer or parameters)
- A static factory method (`newFromXxx`)
- A `buildContract()` method decorated with `[Wrappable(true), Replaceable(true)]`

### 2. Contract Class (extends `BusinessEventsContract`)

Decorated with `[DataContract]`. Contains:
- Private fields for each payload value
- A static factory method and private `initialize()` to populate fields
- Parm methods decorated with `[DataMember]` and `[BusinessEventsDataMember]`

### 3. Trigger Extension

A Chain of Command (CoC) extension on the source table or class that:
1. Checks if the business event is enabled via `BusinessEventsConfigurationReader::isBusinessEventEnabled()`
2. Creates the event via the static factory method
3. Calls `.send()` to fire the event

## Enabling Events

1. Navigate to **System Administration > Setup > Business Events > Business Events Catalog**
2. Find the event by name or category
3. Click **Activate** and configure the endpoint (Service Bus, Event Grid, HTTPS, etc.)

## Model Dependencies

- ApplicationCommon
- ApplicationFoundation
- ApplicationPlatform
- ApplicationSuite
- Ledger

## Solution Structure

The Visual Studio solution (`Projects/AAXBusinessEvents.sln`) organizes each business event into its own project (`.rnrproj`), containing the event class, contract class, and trigger extension.
