# Item Tracking in Business Central: Managing Serial, Lot, and Package Numbers

This documentation provides a comprehensive guide to understanding and implementing item tracking in Microsoft Dynamics 365 Business Central. It covers the architecture, technical components, integration points, and best practices for tracking items using serial numbers, lot numbers, and package numbers throughout the inventory lifecycle.

## Introduction: The Philosophy of Item Tracking

Item tracking in Business Central enables organizations to trace items both backwards and forwards in the supply chain. This functionality is essential for quality assurance, regulatory compliance, product recalls, warranty management, and overall inventory visibility. Business Central's item tracking system is designed around these key principles:

- **Comprehensive Tracking**: Follow items throughout their entire lifecycle from receipt to shipment
- **Flexible Configuration**: Customize tracking requirements by item and transaction type
- **Integration**: Seamlessly connect with inventory, warehouse, manufacturing, and sales processes
- **Auditability**: Maintain a complete history of all tracked items
- **Multi-Dimensional Tracking**: Support for serial numbers, lot numbers, and package numbers


### Business Context

Item tracking addresses critical business needs across various industries:

- **Manufacturing**: Track components through production for quality control and recalls
- **Distribution**: Manage expiration dates and ensure first-in-first-out (FIFO) rotation
- **Regulated Industries**: Maintain compliance with traceability requirements
- **Warranty Management**: Link serial numbers to warranty periods and service history
- **Recall Management**: Quickly identify and locate affected items in the supply chain


## Overall Architecture

Business Central's item tracking architecture is built on a dedicated object structure that maintains relationships between items, tracking numbers, and transactions.

```mermaid
flowchart TD
    subgraph "Setup"
        ITC[Item Tracking Codes]
        ITS[Item Tracking Setup]
        ItemCard[Item Card]
    end

    subgraph "Transaction Processing"
        TL[Transaction Lines]
        ITL[Item Tracking Lines]
        TS[Tracking Specification]
    end

    subgraph "Posting"
        CP[Central Posting]
        RE[Reservation Entries]
    end

    subgraph "History"
        ILE[Item Ledger Entries]
        IER[Item Entry Relation]
        VER[Value Entry Relation]
    end
    
    ITC --> ITS
    ITS --> ItemCard
    ItemCard --> TL
    TL --> ITL
    ITL --> TS
    TS --> RE
    RE --> CP
    CP --> ILE
    CP --> IER
    CP --> VER
    ILE --> IER
    ILE --> VER
```


### Architecture Description

The item tracking architecture in Business Central is designed to maintain a comprehensive record of tracked items throughout their lifecycle:

1. **Setup Layer**: Defines how items are tracked through Item Tracking Codes assigned to items
2. **Transaction Layer**: Captures tracking information during document entry using Item Tracking Lines
3. **Processing Layer**: Manages tracking data in Reservation Entries and temporary Tracking Specifications
4. **History Layer**: Posts and maintains tracking information in Item Ledger Entries with relation tables

This architecture enables seamless tracing of items from receipt through to shipment or consumption, with full visibility of all movements and transactions. The system handles the complexities of tracking across partial receipts, shipments, and inventory adjustments.

## Data Flow Architecture

The following diagram illustrates how item tracking data flows through Business Central:

```mermaid
flowchart TD
    subgraph "Inbound Documents"
        PO[Purchase Orders]
        TR[Transfer Receipts]
        PR[Production Output]
    end
    
    subgraph "Item Tracking Assignment"
        ITL[Item Tracking Lines]
        A[Automatic Assignment]
        M[Manual Entry]
        C[Custom Assignment]
    end
    
    subgraph "Internal Storage"
        RE[Reservation Entries]
        TS[Tracking Specification]
    end
    
    subgraph "Posting Process"
        JP[Journal Posting]
        DP[Document Posting]
    end
    
    subgraph "Posted Entities"
        ILE[Item Ledger Entries]
        IER[Item Entry Relation]
        VER[Value Entry Relation]
    end
    
    subgraph "Outbound Documents"
        SO[Sales Orders]
        TS[Transfer Shipments]
        PC[Production Consumption]
    end
    
    PO --> ITL
    TR --> ITL
    PR --> ITL
    ITL --> A
    ITL --> M
    ITL --> C
    A --> RE
    M --> RE
    C --> RE
    RE --> TS
    TS --> JP
    TS --> DP
    JP --> ILE
    DP --> ILE
    ILE --> IER
    ILE --> VER
    ILE --> SO
    ILE --> TS
    ILE --> PC
```


### Data Flow Description

1. **Inbound Processing**:
    - When items arrive, tracking numbers are assigned via Item Tracking Lines
    - Numbers can be assigned automatically, entered manually, or generated based on custom rules
    - These assignments create Reservation Entry records with a status of "Surplus"
2. **Internal Representation**:
    - Tracking data is stored temporarily in Tracking Specification records
    - Permanent tracking allocations are stored in Reservation Entry records
    - These entries link document lines to specific tracking numbers
3. **Posting Process**:
    - During posting, Codeunit 22 "Item Jnl. - Post Line" creates separate Item Ledger Entries for each unique tracking number
    - Special relation tables maintain the one-to-many relationship between document lines and item ledger entries
4. **Outbound Processing**:
    - When items are shipped or consumed, users select specific tracking numbers from available inventory
    - The system ensures only available tracking numbers can be selected based on FIFO/LIFO rules or specific tracking

## Source Code Architecture

The item tracking functionality is implemented through a series of interrelated objects:

```mermaid
classDiagram
    class ItemTrackingCode {
        +Code : Code[^10]
        +Description : Text[^50]
        +SN Specific Tracking : Boolean
        +SN Warehouse Tracking : Boolean
        +SN Purchase Inbound Tracking : Boolean
        +SN Purchase Outbound Tracking : Boolean
        +SN Sales Inbound Tracking : Boolean
        +SN Sales Outbound Tracking : Boolean
        +Lot Specific Tracking : Boolean
        +Lot Warehouse Tracking : Boolean
        +Lot Purchase Inbound Tracking : Boolean
        +Lot Purchase Outbound Tracking : Boolean
        +Lot Sales Inbound Tracking : Boolean
        +Lot Sales Outbound Tracking : Boolean
        +Package Specific Tracking : Boolean
        +Package Warehouse Tracking : Boolean
    }
    
    class Item {
        +No. : Code[^20]
        +Description : Text[^100]
        +Item Tracking Code : Code[^10]
        +Serial Nos. : Code[^20]
        +Lot Nos. : Code[^20]
    }
    
    class ReservationEntry {
        +Entry No. : Integer
        +Item No. : Code[^20]
        +Source Type : Integer
        +Source Subtype : Integer
        +Source ID : Code[^20]
        +Source Batch Name : Code[^10]
        +Source Prod. Order Line : Integer
        +Source Ref. No. : Integer
        +Serial No. : Code[^50]
        +Lot No. : Code[^50]
        +Package No. : Code[^50]
        +Quantity : Decimal
        +Reservation Status : Enum
    }
    
    class TrackingSpecification {
        +Entry No. : Integer
        +Item No. : Code[^20]
        +Source Type : Integer
        +Source Subtype : Integer
        +Source ID : Code[^20]
        +Source Batch Name : Code[^10]
        +Source Prod. Order Line : Integer
        +Source Ref. No. : Integer
        +Serial No. : Code[^50]
        +Lot No. : Code[^50]
        +Package No. : Code[^50]
        +Quantity : Decimal
    }
    
    class ItemLedgerEntry {
        +Entry No. : Integer
        +Item No. : Code[^20]
        +Document Type : Enum
        +Document No. : Code[^20]
        +Serial No. : Code[^50]
        +Lot No. : Code[^50]
        +Package No. : Code[^50]
        +Quantity : Decimal
    }
    
    class ItemEntryRelation {
        +Entry No. : Integer
        +Source ID : Code[^20]
        +Source Type : Integer
        +Source Subtype : Integer
        +Source Batch Name : Code[^10]
        +Source Prod. Order Line : Integer
        +Source Ref. No. : Integer
        +Item Entry No. : Integer
    }
    
    ItemTrackingCode -- Item : defines tracking rules
    Item -- ReservationEntry : generates
    ReservationEntry -- TrackingSpecification : creates temporary
    TrackingSpecification -- ItemLedgerEntry : posts to
    ItemLedgerEntry -- ItemEntryRelation : links via
```


### Source Code Description

The source code architecture reveals several important aspects:

1. **Item Tracking Code** defines rules for how items are tracked throughout various processes
2. **Item** entity contains tracking configuration and links to numbering series
3. **Reservation Entry** serves as the primary structure for pending tracking numbers
4. **Tracking Specification** provides a temporary structure for processing tracking data
5. **Item Ledger Entry** stores posted tracking information
6. **Item Entry Relation** maintains relationships between documents and item ledger entries

The actual implementation involves several codeunits that handle business logic, most notably:

- Codeunit "Item Tracking Management" (main business logic)
- Codeunit "Item Tracking Data Collection" (collecting tracking information)
- Codeunit 22 "Item Jnl. - Post Line" (posting logic for tracked items)


## Integration Points by Object Type

### Codeunits

#### Codeunit "Item Tracking Management"

This core codeunit manages item tracking logic throughout Business Central.


| Procedure | Description | Parameters |
| :-- | :-- | :-- |
| GetItemTrackingSetup | Retrieves setup of item tracking based on item tracking code | ItemTrackingCode, EntryType, Inbound, ItemTrackingSetup |
| RetrieveConsumpItemTracking | Retrieves consumption item tracking | ItemJnlLine, TempHandlingSpecification |

#### Codeunit "Item Tracking Data Collection" (ID 6501)

This codeunit handles the collection and processing of item tracking data.


| Procedure | Description | Parameters |
| :-- | :-- | :-- |
| AssistEditTrackingNo | Assists with editing tracking numbers | TempTrackingSpecification, SearchForSupply, CurrentSignFactor, LookupMode, MaxQuantity |
| AssistOutBoundBarcodeScannerTrackingNo | Processes barcode scanning for tracking numbers | BarcodeResult, TempTrackingSpecification, SearchForSupply, CurrentSignFactor, LookupMode, MaxQuantity |
| SelectMultipleTrackingNo | Enables selection of multiple tracking numbers | TempTrackingSpecification, MaxQuantity, CurrentSignFactor |
| LookupTrackingAvailability | Looks up availability of tracking numbers | TempTrackingSpecification, LookupMode |

#### Codeunit "Create Reserv. Entry"

This codeunit is responsible for creating reservation entries for item tracking.


| Procedure | Description | Parameters |
| :-- | :-- | :-- |
| CreateReservEntryFor | Creates a reservation entry for a source | SourceType, SourceSubtype, SourceID, SourceBatchName, SourceProdOrderLine, SourceRefNo, Quantity, QuantityBase, ReservEntry |
| CreateReservEntryFrom | Creates a reservation entry from a tracking specification | TrackingSpecification |
| CreateEntry | Creates a reservation entry with item details | ItemNo, VariantCode, LocationCode, Description, ExpectedReceiptDate, ShipmentDate, TransferredFromEntryNo, Status |

### Enums

#### Enum "Item Tracking Type"

Defines the types of item tracking available in Business Central.


| Value | Description |
| :-- | :-- |
| Serial No. | Identifies individual items uniquely |
| Lot No. | Identifies batches or groups of items |
| Package No. | Identifies packages containing multiple items |

### Tables

#### Table "Item Tracking Code"

Defines how items are tracked through the inventory system.


| Field | Type | Description |
| :-- | :-- | :-- |
| Code | Code[^10] | Primary key for the item tracking code |
| Description | Text[^50] | Description of the tracking code |
| SN Specific Tracking | Boolean | Requires specific serial numbers for outbound transactions |
| SN Warehouse Tracking | Boolean | Enables serial number tracking in warehouse activities |
| SN Purchase Inbound Tracking | Boolean | Enables serial number tracking for purchase receipts |
| SN Purchase Outbound Tracking | Boolean | Enables serial number tracking for purchase returns |
| SN Sales Inbound Tracking | Boolean | Enables serial number tracking for sales returns |
| SN Sales Outbound Tracking | Boolean | Enables serial number tracking for sales shipments |
| Lot Specific Tracking | Boolean | Requires specific lot numbers for outbound transactions |
| Lot Warehouse Tracking | Boolean | Enables lot number tracking in warehouse activities |
| Lot Purchase Inbound Tracking | Boolean | Enables lot number tracking for purchase receipts |
| Lot Purchase Outbound Tracking | Boolean | Enables lot number tracking for purchase returns |
| Lot Sales Inbound Tracking | Boolean | Enables lot number tracking for sales returns |
| Lot Sales Outbound Tracking | Boolean | Enables lot number tracking for sales shipments |
| Package Specific Tracking | Boolean | Requires specific package numbers for outbound transactions |
| Package Warehouse Tracking | Boolean | Enables package tracking in warehouse activities |

#### Table "Reservation Entry" (T337)

Stores item tracking numbers for order network entities and non-order network entities.


| Field | Type | Description |
| :-- | :-- | :-- |
| Entry No. | Integer | Primary key for the reservation entry |
| Item No. | Code[^20] | Item number being tracked |
| Source Type | Integer | Type of source document (e.g., Sales Order, Purchase Order) |
| Source Subtype | Integer | Subtype of source document |
| Source ID | Code[^20] | ID of the source document |
| Source Batch Name | Code[^10] | Batch name for journal entries |
| Source Prod. Order Line | Integer | Production order line reference |
| Source Ref. No. | Integer | Reference number within the source document |
| Serial No. | Code[^50] | Serial number for item tracking |
| Lot No. | Code[^50] | Lot number for item tracking |
| Package No. | Code[^50] | Package number for item tracking |
| Quantity | Decimal | Quantity being tracked |
| Reservation Status | Enum | Status of the reservation (e.g., Surplus, Reservation) |

#### Table "Tracking Specification" (T336)

Temporary table used by the Item Tracking Lines page to manage tracking data.


| Field | Type | Description |
| :-- | :-- | :-- |
| Entry No. | Integer | Primary key for the tracking specification |
| Item No. | Code[^20] | Item number being tracked |
| Source Type | Integer | Type of source document |
| Source Subtype | Integer | Subtype of source document |
| Source ID | Code[^20] | ID of the source document |
| Source Batch Name | Code[^10] | Batch name for journal entries |
| Source Prod. Order Line | Integer | Production order line reference |
| Source Ref. No. | Integer | Reference number within the source document |
| Serial No. | Code[^50] | Serial number for item tracking |
| Lot No. | Code[^50] | Lot number for item tracking |
| Package No. | Code[^50] | Package number for item tracking |
| Quantity | Decimal | Quantity being tracked |

#### Table "Item Entry Relation" (T6507)

Links posted document lines to item ledger entries.


| Field | Type | Description |
| :-- | :-- | :-- |
| Entry No. | Integer | Primary key for the relation |
| Source ID | Code[^20] | ID of the source document |
| Source Type | Integer | Type of source document |
| Source Subtype | Integer | Subtype of source document |
| Source Batch Name | Code[^10] | Batch name for journal entries |
| Source Prod. Order Line | Integer | Production order line reference |
| Source Ref. No. | Integer | Reference number within the source document |
| Item Entry No. | Integer | Reference to the item ledger entry |

#### Table "Value Entry Relation" (T6508)

Links invoiced lines to value entries.


| Field | Type | Description |
| :-- | :-- | :-- |
| Value Entry No. | Integer | Reference to the value entry |
| Source RowId | Text | Row ID of the source document |

### Events

Business Central provides several events to extend item tracking functionality:


| Event | When to Use |
| :-- | :-- |
| OnRegisterChangeOnChangeTypeInsertOnBeforeInsertReservEntry | Use when you need to modify reservation entries before they are inserted during item tracking changes |
| OnAfterSetDates | Use when you need to perform actions after dates are set in reservation entries |
| OnBeforeAutoReserveItemTracking | Use to customize the automatic reservation of item tracking numbers |
| OnBeforeCheckItemTrackingInformation | Use to implement custom validation of item tracking information |
| OnBeforeCreateReservEntry | Use to customize the creation of reservation entries |
| OnAfterCopyTrackingFromItemLedgEntry | Use to perform actions after copying tracking from item ledger entries |

### Pages

#### Page "Item Tracking Lines"

This page allows users to assign and view item tracking numbers for document lines.


| Purpose | Description |
| :-- | :-- |
| Main Function | Provides an interface for assigning serial, lot, and package numbers to document lines |
| Usage | Opened from purchase, sales, and inventory documents to manage tracking numbers |
| Key Features | Shows quantity summary, allows manual entry or automatic assignment, displays availability |

#### Page "Item Tracking Code Card"

This page is used to set up and configure item tracking codes.


| Purpose | Description |
| :-- | :-- |
| Main Function | Configures how items are tracked throughout the inventory system |
| Usage | Used during setup to define tracking requirements by item type |
| Key Features | Controls inbound/outbound tracking, specific tracking, warehouse handling |

#### Page "Item Tracking Summary"

Shows a summary of item tracking information for a document line.


| Purpose | Description |
| :-- | :-- |
| Main Function | Provides an overview of tracking numbers and quantities |
| Usage | Used to review tracking assignments before posting |
| Key Features | Shows totals by tracking number type, highlights discrepancies |

## How to Extend Item Tracking

When extending item tracking functionality in Business Central, follow these best practices to ensure your customizations work seamlessly with Microsoft's standard functionality.

### Adding Custom Fields to Item Tracking

To add custom information to tracked items:

1. **Extend the Data Model**:
    - Create table extensions for Tracking Specification, Reservation Entry, and Item Ledger Entry
    - Add the same custom fields to all tables to maintain consistency
2. **Extend the User Interface**:
    - Create page extensions for Item Tracking Lines and Item Ledger Entries
    - Add the custom fields to provide input and display capabilities
3. **Implement Data Flow**:
    - Subscribe to the `OnRegisterChangeOnChangeTypeInsertOnBeforeInsertReservEntry` event to capture values from UI
    - Subscribe to the `OnAfterSetDates` event to transfer values to reservation entries
    - Use event subscribers to flow values to Item Ledger Entries during posting

### Creating Custom Item Tracking Automation

For automatic assignment of tracking numbers:

1. **Create a codeunit for your automation logic**
2. **Use the `CreateReservEntryFor` and `CreateEntry` methods to create tracking entries programmatically**
3. **Implement your business rules to determine which tracking numbers to assign**

### Integrating with External Systems

To integrate item tracking with external systems:

1. **Create API pages to expose item tracking data**
2. **Implement mappings between external identifiers and Business Central tracking numbers**
3. **Use `CreateReservEntryFor` and `CreateEntry` to create tracking records from external data**

### Best Practices

- **Respect the Item Tracking Code Configuration**: Always check the item tracking code settings before applying custom logic
- **Maintain Data Integrity**: Ensure tracking numbers are consistently applied across all related documents
- **Consider Performance**: Item tracking can impact performance with large volumes, optimize your extensions accordingly
- **Use Events Over Direct Modifications**: Extend via event subscribers rather than modifying standard objects
- **Test with Partial Transactions**: Ensure your extensions handle partial receipts and shipments correctly


## Code Examples

### Example 1: Automatic Assignment of Lot Numbers Based on Expiration Date

This example demonstrates how to automatically assign lot numbers to items on a sales order based on their expiration dates (FEFO - First Expired, First Out).

```al
codeunit 50100 "Auto Assign Lot Numbers"
{
    // Assigns lot numbers automatically based on expiration date
    procedure AssignLotNumbersToSalesLine(var SalesLine: Record "Sales Line")
    var
        Item: Record Item;
        ItemTrackingCode: Record "Item Tracking Code";
        ItemLedgerEntry: Record "Item Ledger Entry";
        TempReservationEntry: Record "Reservation Entry" temporary;
        TempTrackingSpecification: Record "Tracking Specification" temporary;
        CreateReservEntry: Codeunit "Create Reserv. Entry";
        QtyToAssign: Decimal;
        QtyToAssignBase: Decimal;
    begin
        // Check if item requires lot tracking
        if not Item.Get(SalesLine."No.") then
            exit;
            
        if Item."Item Tracking Code" = '' then
            exit;
            
        if not ItemTrackingCode.Get(Item."Item Tracking Code") then
            exit;
            
        if not ItemTrackingCode."Lot Sales Outbound Tracking" then
            exit;
            
        // Calculate quantity to assign
        QtyToAssign := SalesLine.Quantity;
        QtyToAssignBase := SalesLine."Quantity (Base)";
        
        // Find available lot numbers sorted by expiration date (FEFO)
        ItemLedgerEntry.Reset();
        ItemLedgerEntry.SetCurrentKey("Item No.", "Expiration Date");
        ItemLedgerEntry.SetRange("Item No.", SalesLine."No.");
        ItemLedgerEntry.SetRange("Location Code", SalesLine."Location Code");
        ItemLedgerEntry.SetFilter("Remaining Quantity", '>0');
        ItemLedgerEntry.SetRange(Open, true);
        
        // Process each available lot
        if ItemLedgerEntry.FindSet() then
            repeat
                if QtyToAssign <= 0 then
                    break;
                    
                // Calculate quantity to take from this lot
                QtyToAssign := Minimum(QtyToAssign, ItemLedgerEntry."Remaining Quantity");
                QtyToAssignBase := Minimum(QtyToAssignBase, ItemLedgerEntry."Remaining Quantity");
                
                if QtyToAssign > 0 then begin
                    // Prepare reservation entry with lot number
                    TempReservationEntry.Init();
                    TempReservationEntry."Lot No." := ItemLedgerEntry."Lot No.";
                    
                    // Create reservation entry for the sales line
                    CreateReservEntry.CreateReservEntryFor(
                        Database::"Sales Line", 
                        SalesLine."Document Type".AsInteger(), 
                        SalesLine."Document No.", 
                        '', 
                        0,
                        SalesLine."Line No.", 
                        SalesLine."Qty. per Unit of Measure", 
                        QtyToAssign, 
                        QtyToAssignBase, 
                        TempReservationEntry
                    );
                    
                    // Create reservation from item ledger entry
                    TempTrackingSpecification.InitTrackingSpecification(
                        Database::"Item Ledger Entry", 
                        0, 
                        '', 
                        '', 
                        0, 
                        ItemLedgerEntry."Entry No.",
                        ItemLedgerEntry."Variant Code", 
                        ItemLedgerEntry."Location Code", 
                        ItemLedgerEntry."Qty. per Unit of Measure"
                    );
                    TempTrackingSpecification.CopyTrackingFromItemLedgEntry(ItemLedgerEntry);
                    CreateReservEntry.CreateReservEntryFrom(TempTrackingSpecification);
                    
                    // Update remaining quantity to assign
                    QtyToAssign := QtyToAssign - ItemLedgerEntry."Remaining Quantity";
                    QtyToAssignBase := QtyToAssignBase - ItemLedgerEntry."Remaining Quantity";
                end;
            until ItemLedgerEntry.Next() = 0;
            
        // Error if not all quantity could be assigned
        if QtyToAssign > 0 then
            Error('Could not assign lot numbers for the full quantity. Available quantity is insufficient.');
    end;
    
    local procedure Minimum(Value1: Decimal; Value2: Decimal): Decimal
    begin
        if Value1 < Value2 then
            exit(Value1);
        exit(Value2);
    end;
}
```


### Example 2: Adding Custom Field to Item Tracking

This example demonstrates how to add a "Quality Score" field to item tracking.

```al
// 1. Table Extension for Tracking Specification
tableextension 50100 "Tracking Specification Ext" extends "Tracking Specification"
{
    fields
    {
        field(50100; "Quality Score"; Decimal)
        {
            Caption = 'Quality Score';
            MinValue = 0;
            MaxValue = 100;
            DecimalPlaces = 0:0;
        }
    }
}

// 2. Table Extension for Reservation Entry
tableextension 50101 "Reservation Entry Ext" extends "Reservation Entry"
{
    fields
    {
        field(50100; "Quality Score"; Decimal)
        {
            Caption = 'Quality Score';
            MinValue = 0;
            MaxValue = 100;
            DecimalPlaces = 0:0;
        }
    }
}

// 3. Table Extension for Item Ledger Entry
tableextension 50102 "Item Ledger Entry Ext" extends "Item Ledger Entry"
{
    fields
    {
        field(50100; "Quality Score"; Decimal)
        {
            Caption = 'Quality Score';
            MinValue = 0;
            MaxValue = 100;
            DecimalPlaces = 0:0;
        }
    }
}

// 4. Page Extension for Item Tracking Lines
pageextension 50100 "Item Tracking Lines Ext" extends "Item Tracking Lines"
{
    layout
    {
        addafter("Lot No.")
        {
            field("Quality Score"; Rec."Quality Score")
            {
                ApplicationArea = All;
                ToolTip = 'Specifies the quality score for this lot or serial number.';
            }
        }
    }
}

// 5. Codeunit to handle data flow
codeunit 50101 "Item Tracking Custom Fields"
{
    // Store values when user updates tracking lines
    [EventSubscriber(ObjectType::Page, Page::"Item Tracking Lines", 'OnRegisterChangeOnChangeTypeInsertOnBeforeInsertReservEntry', '', false, false)]
    local procedure OnRegisterChange(var ReservEntry: Record "Reservation Entry"; var TrackingSpecification: Record "Tracking Specification" temporary)
    begin
        ReservEntry."Quality Score" := TrackingSpecification."Quality Score";
    end;
    
    // Transfer values to item ledger entries during posting
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Item Jnl.-Post Line", 'OnAfterInitItemLedgEntry', '', false, false)]
    local procedure OnAfterInitItemLedgEntry(var NewItemLedgEntry: Record "Item Ledger Entry"; var ItemJournalLine: Record "Item Journal Line"; var ItemLedgEntryNo: Integer)
    var
        ReservEntry: Record "Reservation Entry";
    begin
        ReservEntry.Reset();
        ReservEntry.SetRange("Source Type", Database::"Item Journal Line");
        ReservEntry.SetRange("Source ID", ItemJournalLine."Journal Template Name");
        ReservEntry.SetRange("Source Batch Name", ItemJournalLine."Journal Batch Name");
        ReservEntry.SetRange("Source Ref. No.", ItemJournalLine."Line No.");
        if ReservEntry.FindFirst() then
            NewItemLedgEntry."Quality Score" := ReservEntry."Quality Score";
    end;
}
```


### Example 3: Bulk Import of Tracking Numbers from Excel

This example shows how to import tracking numbers from Excel.

```al
codeunit 50102 "Import Tracking Numbers"
{
    procedure ImportTrackingNumbersFromExcel(DocumentType: Enum "Purchase Document Type"; DocumentNo: Code[^20])
    var
        PurchaseLine: Record "Purchase Line";
        ExcelBuffer: Record "Excel Buffer" temporary;
        TempBlob: Codeunit "Temp Blob";
        FileMgt: Codeunit "File Management";
        InsStr: InStream;
        OutStr: OutStream;
        FileName: Text;
        SheetName: Text;
        RowNo: Integer;
        ColNo: Integer;
        ItemNo: Code[^20];
        SerialNo: Code[^50];
        LotNo: Code[^50];
        Quantity: Decimal;
    begin
        // Get the Excel file
        FileName := FileMgt.BLOBImportWithFilter(TempBlob, 'Import Tracking Numbers', '', 'Excel Files (*.xlsx)|*.xlsx', '*.xlsx');
        if FileName = '' then
            exit;
            
        // Load Excel into buffer
        TempBlob.CreateInStream(InsStr);
        SheetName := ExcelBuffer.SelectSheetsNameStream(InsStr);
        ExcelBuffer.OpenBookStream(InsStr, SheetName);
        ExcelBuffer.ReadSheet();
        
        // Process rows in the Excel file
        for RowNo := 2 to ExcelBuffer.GetNumberOfRows() do begin  // Start from row 2 to skip header
            // Read data from Excel
            ItemNo := GetExcelCellValue(ExcelBuffer, RowNo, 1);
            SerialNo := GetExcelCellValue(ExcelBuffer, RowNo, 2);
            LotNo := GetExcelCellValue(ExcelBuffer, RowNo, 3);
            Evaluate(Quantity, GetExcelCellValue(ExcelBuffer, RowNo, 4));
            
            // Find purchase line
            PurchaseLine.Reset();
            PurchaseLine.SetRange("Document Type", DocumentType);
            PurchaseLine.SetRange("Document No.", DocumentNo);
            PurchaseLine.SetRange("No.", ItemNo);
            
            if PurchaseLine.FindFirst() then
                AssignItemTracking(PurchaseLine, SerialNo, LotNo, Quantity);
        end;
        
        Message('Tracking numbers imported successfully.');
    end;
    
    local procedure GetExcelCellValue(var ExcelBuffer: Record "Excel Buffer" temporary; RowNo: Integer; ColNo: Integer): Text
    begin
        ExcelBuffer.Reset();
        if ExcelBuffer.Get(RowNo, ColNo) then
            exit(ExcelBuffer."Cell Value as Text");
        exit('');
    end;
    
    local procedure AssignItemTracking(PurchaseLine: Record "Purchase Line"; SerialNo: Code[^50]; LotNo: Code[^50]; Quantity: Decimal)
    var
        TempReservationEntry: Record "Reservation Entry" temporary;
        CreateReservEntry: Codeunit "Create Reserv. Entry";
        QtyToAssignBase: Decimal;
    begin
        // Calculate base quantity
        QtyToAssignBase := Quantity * PurchaseLine."Qty. per Unit of Measure";
        
        // Set up tracking information
        TempReservationEntry.Init();
        TempReservationEntry."Serial No." := SerialNo;
        TempReservationEntry."Lot No." := LotNo;
        
        // Create reservation entry for the purchase line
        CreateReservEntry.CreateReservEntryFor(
            Database::"Purchase Line", 
            PurchaseLine."Document Type".AsInteger(), 
            PurchaseLine."Document No.", 
            '', 
            0,
            PurchaseLine."Line No.", 
            PurchaseLine."Qty. per Unit of Measure", 
            Quantity, 
            QtyToAssignBase, 
            TempReservationEntry
        );
        
        // Create the entry with status Surplus
        CreateReservEntry.CreateEntry(
            PurchaseLine."No.", 
            PurchaseLine."Variant Code", 
            PurchaseLine."Location Code", 
            PurchaseLine.Description, 
            0D,
            PurchaseLine."Expected Receipt Date", 
            0, 
            Enum::"Reservation Status"::Surplus
        );
    end;
}
```


## Troubleshooting Guide

### Performance Bottlenecks with Large Volumes

#### Issue: Slow Loading of Item Tracking Lines Page

**Symptoms:**

- The Item Tracking Lines page takes several seconds to load
- Performance degrades as inventory volume increases

**Solutions:**

1. **Optimize Database Indexes:**
    - Ensure proper indexes on Serial No., Lot No., and Package No. fields in the Item Ledger Entry table
    - Consider adding custom indexes on frequently filtered fields
2. **Implement Paging for Large Datasets:**
    - Modify page extensions to load data in smaller chunks
    - Use filters to limit the number of records displayed
3. **Regular Maintenance:**
    - Regularly run the Adjust Cost - Item Entries batch job to keep item ledger entries optimized
    - Archive old tracking information when no longer needed

#### Issue: Slow Posting of Documents with Many Tracking Lines

**Symptoms:**

- Posting takes excessive time when many tracking lines exist
- System resources spike during posting

**Solutions:**

1. **Batch Processing:**
    - Split large documents into smaller batches for posting
    - Use background posting for large documents
2. **Optimize Custom Code:**
    - Ensure any custom code in item tracking events is efficient
    - Avoid nested loops and redundant database queries
3. **Hardware Resources:**
    - Consider upgrading server resources if tracking volumes are consistently high

### Data Integrity Issues

#### Issue: Mismatched Tracking Quantities

**Symptoms:**

- Quantities in Item Tracking Lines don't match document lines
- Error messages about undefined tracking quantities

**Solutions:**

1. **Verify Tracking Setup:**
    - Check that item tracking codes are correctly configured
    - Ensure tracking is required for appropriate transaction types
2. **Use Built-in Validation:**
    - Always use the Item Tracking Lines page to assign tracking numbers
    - Let the system validate quantities before posting
3. **Recovery Procedure:**
    - Use the Item Tracking Summary page to identify discrepancies
    - Adjust tracking lines to match document quantities

#### Issue: Orphaned Tracking Entries

**Symptoms:**

- Tracking numbers appear reserved but are not actually available
- Inventory is blocked unnecessarily

**Solutions:**

1. **Run Reservation Cleanup:**
    - Use the Delete Expired Reservation Entries batch job
    - Set up a recurring task to clean up orphaned entries
2. **Manual Correction:**
    - Identify orphaned entries using the Reservation Entries page
    - Delete or correct entries as needed
3. **Preventive Measures:**
    - Always use the Cancel function rather than deleting documents
    - Complete document workflows properly

### Common Implementation Mistakes

#### Mistake: Changing Item Tracking Code After Transactions

This is problematic because the tracking code defines how items are tracked throughout their lifecycle.

**Prevention/Solution:**

- Plan tracking requirements carefully before setting up items
- If changes are needed, create new items with the correct tracking code
- Transfer inventory from old items to new items with the correct tracking


#### Mistake: Custom Code Bypassing Standard Item Tracking Logic

This can lead to data integrity issues and unpredictable behavior.

**Prevention/Solution:**

- Always use the standard API (CreateReservEntry) to create tracking entries
- Subscribe to events rather than replacing standard code
- Test custom code thoroughly with edge cases


#### Mistake: Inadequate User Training on Item Tracking

Users may struggle with tracking if they don't understand the principles.

**Prevention/Solution:**

- Provide clear documentation and training for users
- Create standard procedures for common tracking scenarios
- Configure the system to minimize manual input where possible


### Diagnostic Tools and Techniques

#### Built-in Diagnostics

1. **Item Tracking Summary Page:**
    - Shows an overview of tracking assignments
    - Highlights discrepancies between document and tracking quantities
2. **Reservation Entries Page:**
    - Shows all reservation entries, including tracking information
    - Useful for identifying issues with specific tracking numbers
3. **Navigate Function:**
    - Use the Navigate function on item ledger entries to trace tracking numbers
    - Shows all related documents and entries

#### Advanced Diagnostics

1. **Telemetry Analysis:**
    - Enable performance telemetry for item tracking operations
    - Analyze timings to identify bottlenecks
2. **SQL Query Analysis:**
    - Analyze query execution plans for item tracking operations
    - Identify slow-performing queries and optimize indexes
3. **Custom Diagnostic Reports:**
    - Create custom reports to validate tracking data integrity
    - Compare tracking numbers across related documents and entries

## Quick Start Implementation Guide

### Setting Up Item Tracking in Business Central

Follow these steps to implement basic item tracking for your inventory:

#### 1. Create Item Tracking Codes

1. Choose the search 🔎 icon, enter **Item Tracking Codes**, and then choose the related link
2. Choose the **New** action
3. Fill in the following fields:
    - **Code**: A unique identifier (e.g., "LOTALL")
    - **Description**: Descriptive name (e.g., "Lot tracking for all transactions")
4. On the **Lot No.** FastTab, set the following:
    - Enable **Lot Specific Tracking** if you need to track specific lots throughout their lifecycle
    - Enable **Lot Purchase Inbound Tracking** to require lot numbers when receiving items
    - Enable **Lot Sales Outbound Tracking** to require lot numbers when shipping items
5. Configure similar settings on the **Serial No.** and **Package Tracking** FastTabs as needed
6. Choose **OK** to save the tracking code

#### 2. Assign Tracking Codes to Items

1. Choose the search 🔎 icon, enter **Items**, and then choose the related link
2. Open an item card for an item you want to track
3. On the **Item Tracking** FastTab, set:
    - **Item Tracking Code**: Select the code you created
    - **Serial Nos.**: Optional - Select a number series for automatic serial number assignment
    - **Lot Nos.**: Optional - Select a number series for automatic lot number assignment
4. Choose **OK** to save the item

#### 3. Use Item Tracking in Transactions

**For Purchase Orders:**

1. Create a new purchase order
2. Add a line with your tracked item
3. Select the line, then choose **Line** → **Item Tracking Lines**
4. Assign lot or serial numbers:
    - For automatic assignment: Choose **Functions** → **Assign Lot No.** or **Assign Serial No.**
    - For manual entry: Enter numbers directly in the tracking lines
5. Close the tracking page and post the document

**For Sales Orders:**

1. Create a new sales order
2. Add a line with your tracked item
3. Select the line, then choose **Line** → **Item Tracking Lines**
4. Select the lot or serial numbers to ship
5. Close the tracking page and post the document

#### 4. Basic AL Template for Item Tracking Extension

Use this template to create a simple extension that adds custom functionality to item tracking:

```al
// Custom Item Tracking Extension
// Add a "Manufacturing Date" field to item tracking

// Table Extensions
tableextension 50100 "Tracking Specification Ext" extends "Tracking Specification"
{
    fields
    {
        field(50100; "Manufacturing Date"; Date)
        {
            Caption = 'Manufacturing Date';
            DataClassification = CustomerContent;
        }
    }
}

tableextension 50101 "Reservation Entry Ext" extends "Reservation Entry"
{
    fields
    {
        field(50100; "Manufacturing Date"; Date)
        {
            Caption = 'Manufacturing Date';
            DataClassification = CustomerContent;
        }
    }
}

tableextension 50102 "Item Ledger Entry Ext" extends "Item Ledger Entry"
{
    fields
    {
        field(50100; "Manufacturing Date"; Date)
        {
            Caption = 'Manufacturing Date';
            DataClassification = CustomerContent;
        }
    }
}

// Page Extension
pageextension 50100 "Item Tracking Lines Ext" extends "Item Tracking Lines"
{
    layout
    {
        addafter("Expiration Date")
        {
            field("Manufacturing Date"; Rec."Manufacturing Date")
            {
                ApplicationArea = All;
                ToolTip = 'Specifies the date when the item was manufactured.';
            }
        }
    }
}

// Codeunit for Event Subscribers
codeunit 50100 "Item Tracking Custom Fields"
{
    [EventSubscriber(ObjectType::Page, Page::"Item Tracking Lines", 'OnRegisterChangeOnChangeTypeInsertOnBeforeInsertReservEntry', '', false, false)]
    local procedure OnRegisterChange(var ReservEntry: Record "Reservation Entry"; var TrackingSpecification: Record "Tracking Specification" temporary)
    begin
        ReservEntry."Manufacturing Date" := TrackingSpecification."Manufacturing Date";
    end;
    
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Create Reserv. Entry", 'OnAfterSetDates', '', false, false)]
    local procedure OnAfterSetDates(var ReservEntry: Record "Reservation Entry"; var TrackingSpecification: Record "Tracking Specification")
    begin
        ReservEntry."Manufacturing Date" := TrackingSpecification."Manufacturing Date";
    end;
    
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Item Jnl.-Post Line", 'OnAfterInitItemLedgEntry', '', false, false)]
    local procedure OnAfterInitItemLedgEntry(var NewItemLedgEntry: Record "Item Ledger Entry"; var ItemJournalLine: Record "Item Journal Line"; var ItemLedgEntryNo: Integer)
    var
        ReservEntry: Record "Reservation Entry";
    begin
        ReservEntry.Reset();
        ReservEntry.SetRange("Source Type", Database::"Item Journal Line");
        ReservEntry.SetRange("Source ID", ItemJournalLine."Journal Template Name");
        ReservEntry.SetRange("Source Batch Name", ItemJournalLine."Journal Batch Name");
        ReservEntry.SetRange("Source Ref. No.", ItemJournalLine."Line No.");
        if ReservEntry.FindFirst() then
            NewItemLedgEntry."Manufacturing Date" := ReservEntry."Manufacturing Date";
    end;
}
```


## Related Topics for Further Research

- **Warehouse Management Integration with Item Tracking**: Explore how item tracking integrates with directed put-away and pick operations
- **Manufacturing and Item Tracking**: Investigate tracking components through the production process
- **Assembly Management and Item Tracking**: Research how tracking applies to assembly orders and components
- **Barcode Scanning Integration**: Explore how to integrate barcode scanning with item tracking for more efficient operations
- **Advanced Reservation Systems**: Dive deeper into the reservation system and its relationship with item tracking
- **Cross-Reference Item Tracking**: Study how to link internal tracking numbers with vendor and customer tracking numbers
- **Costing Implications of Item Tracking**: Investigate how item tracking affects costing methods and item value entries
- **Item Tracking in Service Management**: Explore tracking items in service orders and service items
- **Regulatory Compliance with Item Tracking**: Research industry-specific compliance requirements and how to meet them using item tracking
- **Performance Optimization for Large-Scale Deployments**: Advanced techniques for optimizing performance with high-volume item tracking

* Sources

[^1]: https://bcdocs.staedean.com/md/en-US/inventory-how-setup-item-tracking.md

[^2]: https://www.azurecurve.co.uk/2024/03/new-functionality-in-microsoft-dynamics-365-business-central-2024-wave-1-inventory-package-numbers-work-like-item-tracking-dimensions/

[^3]: https://ebs.com.au/blog/lot-tracking-microsoft-business-central

[^4]: https://learn.microsoft.com/en-us/dynamics365/business-central/design-details-item-tracking-design

[^5]: https://community.dynamics.com/forums/thread/details/?threadid=eadf217b-2181-48d1-8815-278a43c180d1

[^6]: https://learn.microsoft.com/en-us/dynamics365/business-central/application/base-application/codeunit/microsoft.inventory.tracking.item-tracking-data-collection

[^7]: https://www.sauravdhyani.com/2020/09/enums-available-in-business-central.html

[^8]: https://www.linkedin.com/pulse/business-central-add-custom-fields-item-tracking-lines-anil-kumar

[^9]: https://dynamicscommunities.com/ug/dynamics-business-central-nav-ug/how-to-set-up-item-tracking-codes-in-business-central/

[^10]: https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/developer/devenv-request-pages

[^11]: https://learn.microsoft.com/en-us/dynamics365/business-central/inventory-how-setup-item-tracking

[^12]: https://ivansingleton.dev/bulk-serial-and-lot-number-import-for-transfer-orders-excel-as-a-bridge/

[^13]: https://learn.microsoft.com/en-us/dynamics365/business-central/inventory-how-work-item-tracking

[^14]: https://www.xpandsoftware.com/blogs/business-central/whats-new-in-dynamics-365-business-central-2024-release-wave-2

[^15]: https://yzhums.com/17065/

[^16]: https://probitas.co.uk/2021/07/30/exploring-item-tracking-functionality-in-business-central/

[^17]: https://learn.microsoft.com/en-us/dynamics365/business-central/design-details-item-tracking-posting-structure

[^18]: https://yzhums.com/7943/

[^19]: https://www.linkedin.com/pulse/how-create-reservations-item-tracking-from-al-flemming-bakkensen-6ul2f

[^20]: https://learn.microsoft.com/en-us/dynamics365/business-central/application/base-application/codeunit/microsoft.inventory.tracking.item-tracking-management

[^21]: https://community.dynamics.com/blogs/post/?postid=66144cba-9008-ef11-a73d-000d3a7cd297

[^22]: https://www.youtube.com/watch?v=e-tR9EBGsiA

[^23]: https://usedynamics.com/business-central/inventory/item-tracking-for-lot-serial/

[^24]: https://yzhums.com/56472/

[^25]: https://community.dynamics.com/forums/thread/details/?threadid=377e8988-be05-4d29-8227-efec2c18327a

[^26]: https://usedynamics.com/business-central/inventory/item-tracking-on-shipping/

[^27]: https://saranilango.hashnode.dev/optimizing-performance-in-dynamics-365-business-central-application-areas

[^28]: https://stoneridgesoftware.com/2024-wave-2-release-plan-for-dynamics-365-business-central-10-features-to-get-excited-about/

[^29]: https://community.dynamics.com/forums/thread/details/?threadid=a1752fe1-bc1a-4600-9b3c-f77c4f082879

[^30]: https://www.fenwick.com.au/solutions/easy-item-tracking/

[^31]: https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/performance/performance-work-perf-problem

[^32]: https://bondconsultingservices.com/2023/05/01/business-central-item-journal-with-item-tracked-inventory/

[^33]: https://yzhums.com/11544/

[^34]: https://docs.cosmoconsult.com/business-central/data-integration-framework/mapping/mapping-codeunits.html

[^35]: https://simplanova.com/blog/business-central-enums-use/

[^36]: https://www.olofsimren.com/add-fields-to-the-item-tracking-lines/

[^37]: https://www.youtube.com/watch?v=T-GWbWyq3PI

[^38]: https://yzhums.com/4809/

[^39]: https://yzhums.com/wp-content/uploads/2023/12/ItemTrackingTechWhitePaper.pdf

[^40]: https://mybusinesscentraldiary.wordpress.com/tag/enum/

[^41]: https://stoneridgesoftware.com/item-tracking-capabilities-in-dynamics-365-business-central-and-probatch/

[^42]: https://www.youtube.com/watch?v=UDLxCJxfM80

[^43]: https://community.dynamics.com/forums/thread/details/?threadid=faba79ee-4a9a-4443-a7cc-d00dfd22be4e

[^44]: https://learn.microsoft.com/en-us/dynamics365/business-central/setup

[^45]: https://community.dynamics.com/forums/thread/details/?threadid=87fbe7b5-d471-ee11-9ae7-6045bda7c23b

[^46]: https://skanderby.dk/wp-content/uploads/2025/02/Business-Central-Capability-guide-2025.pdf

[^47]: https://dmsiworks.com/blog/streamlining-compliance-lot-serial-number-tracking-in-business-central

[^48]: https://yzhums.com/2844/

[^49]: https://www.youtube.com/watch?v=dtMYnzetYx0

[^50]: https://community.dynamics.com/blogs/post/?postid=1afe35eb-09f1-ee11-a73d-0022484f4b6d

[^51]: https://github.com/JalmarazMartn/ALBC365ItemTrackingNewField

[^52]: https://community.dynamics.com/forums/thread/details/?threadid=0d81d4aa-70cf-ee11-9079-00224827e8f9

[^53]: https://www.dynamicsuser.net/t/item-tracking-lines/16954

[^54]: https://community.dynamics.com/forums/thread/details/?threadid=ffcc8d38-18a5-457d-a3ea-cf0ed905d380

[^55]: https://taskletfactory.com/media/pslbncyu/mobile-wms-user-guide-microsoft-dynamics-365-business-central-version-5-19.pdf

[^56]: https://community.dynamics.com/forums/thread/details/?threadid=30d10f1e-b80f-4aed-932d-e02349bda5d3

[^57]: https://www.youtube.com/watch?v=kBFlCqVW0a4
