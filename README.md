# Utility Tracker: Power Platform Project

A robust Power Platform solution designed to manage and track utility project entries, leveraging Power Apps for the user interface, SharePoint for data storage, and Power Automate for background data processing and reporting optimization.

## 🏗️ Architecture

The application follows a tiered architecture to ensure data integrity and user efficiency:

1.  **Frontend (Power Apps):** A canvas app providing a streamlined interface for data entry and project browsing. It uses complex Power Fx logic for real-time data lookups and multi-entity patching.
2.  **Backend (SharePoint Online):**
    *   **Mapping Table:** Serves as the Master Data source containing unique utility IDs and associated metadata (Client Name, Site Name, etc.).
    *   **Tracking List:** The primary transactional list where user entries are stored.
3.  **Automation Layer (Power Automate):** Background flows that enrich data by denormalizing complex SharePoint lookups into flat text strings, optimizing the dataset for Power BI.

---

## 🔄 Application Flow

1.  **Selection:** In the **New Entry Screen**, the user selects one or multiple "Unique IDs" from a combo box connected to the *Mapping Table*.
2.  **Auto-Fill:** The app uses the `Concat` function to walk through the selected items and provide a live preview of the associated Client Names and Sites.
3.  **Submission:** On "Save", a `Patch` command shapes and sends the data to the *Tracking List*, handling multi-value lookups and boolean choice fields simultaneously.
4.  **Enrichment:** The creation of a new item triggers a **Power Automate Flow**. The flow loops through the selected IDs, retrieves the full metadata from the *Mapping Table*, and joins these values into semi-colon separated strings.
5.  **Consumption:** The **Home Screen** displays the enriched data in a searchable gallery, allowing users to filter by Client Name using delegation-friendly logic.

---

## 📱 Power App UI & Functionality

![Home Screen](<UI ( Power App )/Meeting Details.png>)

### 1. Home Screen (`scrHome`)
*   **Search & Filter:** A dynamic search bar using the `in` operator allows users to quickly find entries by Client Name.
*   **View Toggling:** A variable-driven UI (`gView`) toggles between "About" and "Meeting" detail galleries without needing separate screens.
*   **Navigation:** Dedicated buttons to initiate a "New" entry or view project documentation.

### 2. New Entry Screen

![New Entry Page](<UI ( Power App )/New Entry Page.png>)

*   **Multi-Select Combo Box:** Allows users to pick multiple Master Data IDs.
*   **Live Metadata Preview:** Uses Power Fx to display related information (Client, Site, Category) before the user even hits save.
*   **Complex Patching:**
    ```powerfx
    Patch('Tracking List', Defaults('Tracking List'), {
      'Unique ID': ForAll(cmb_UniqueID.SelectedItems, { Id: ID, Value: Text('Unique ID') }),
      'Status': drp_Status.Selected,
      'Included in Project Slide': { Value: If(tgl_Slide.Value, "Yes", "No") }
    })
    ```

---

## ⚙️ Power Automate: Tasks & Future Roadmap

![Power Automate Flow](<Power Automate/Power Automate_ Bring Lookup columns.png>)

### Current Tasks
*   **Data Denormalization:** Resolves the "Multi-Value Lookup" limitation in SharePoint by flattening arrays into text strings. This enables simple string-based filtering in Power BI without complex DAX or Power Query transformations.
*   **Incremental ID Assignment:** Automatically assigns a record ID to each entry, ensuring precise mapping with the unique ID in the database.
*   **Mapping Table Synchronization:** Automated updates to the Mapping Table to ensure master data remains current and synchronized with user entries.
*   **Loop Prevention:** The flow is configured to trigger only on *Create*, ensuring that the flow's own "Update" action doesn't cause an infinite loop.

### Future Roadmap

![Getting Unique IDs](<Power Automate/Future Ideas/Getting Unique Power Automate.png>)

*   **Latest Comment Retrieval:** Implementation of logic to parse appended "Version" or "Comment" fields and extract only the most recent entry for summary reporting.
*   **Data Normalization (Row Splitting):** Development of logic to handle multi-value entries by splitting them into individual records (one record per combination) to enhance relational reporting.
*   **Error Handling:** Adding "Try-Catch" scopes to the flow to log failures, provide alerts, and notify administrators.

---

## 📊 Dataverse Scope: Scaling for the Future

While SharePoint provides a quick-start backend, migrating to **Microsoft Dataverse** is the strategic next step for scaling this solution:

1.  **Relational Integrity:** Dataverse supports true many-to-many relationships, replacing the need for complex multi-value lookup columns and manual denormalization flows.
2.  **Power BI Optimization:** Dataverse provides a native DirectQuery connection and "Dataverse Choice" labels, significantly improving report performance and eliminating the 2,000-row delegation limit found in SharePoint.
3.  **Scalability:** As the "Tracking List" grows into the tens of thousands of rows, Dataverse's indexing and advanced query capabilities ensure the app remains performant.
4.  **Security:** Move from list-level permissions to granular row-level and field-level security, ensuring users only see the data relevant to their role.
