## 3.2 Data Product Consumption in Calculation Views

In this exercise we will reverse the direction of sharing. Instead of sharing a calculation view into the context of SAP Business Data Cloud, we are now sharing SAP Datasphere Data Products into SAP HANA Cloud. 

In particular, we will build calculation views on a shared Data Product to illustrate how shared Data Products can be easily enhanced with calculation views. To achieve this, we integrate the Data Product *Customer* into our existing star join *s_products*.

### 3.2.1 Background: How Sharing and Installing a Data Product in SAP HANA Cloud Was Prepared

To avoid a confusing multiple sharing of the same Data Product you will not share the Data Product yourself. Instead the Data Product *Customer* has already been installed for you. 

> **Note:** The Data Product *Customer* has already been shared and installed for you.
> The following sub-sections document how this was done — for reference only.
> No action required on your part.

Here is how this was achieved:

#### 3.2.1.1 Sharing Data Product *Customer* to SAP HANA Cloud

In SAP Business Data Cloud Cockpit we shared the Data Product *Customer*:

We used *Search* under item *Catalog & Marketplace* to search for relevant Data Products:

![search Data Products](./screenshots/dataProducts/search.png)

Searching for the term *customer* (1.) we found a matching Data Product and displayed more details about it by clicking on it (2.)

![show Data Product details](./screenshots/dataProducts/searchCustomer.png)

The Data Product fits our needs, so we clicked on *Share*.

![share](./screenshots/dataProducts/share.png)

We selected the target system (1.) and clicked on *Share* (2.).

![target system](./screenshots/dataProducts/targetSystem.png)

We closed the dialog after the Data Product had been shared successfully.

![success](./screenshots/dataProducts/successfullyShared.png)

#### 3.2.1.2 Installing the Data Product *Customer* in SAP HANA Cloud

In SAP HANA Cloud Central we installed the Data Product *Customer*:

Under tab *Data Products* we used the three dots (1.) and clicked on *Install* (2.) to install the Data Product.

![install](./screenshots/dataProducts/install.png)

We confirmed the information dialog with *Install*.

![accept and install](./screenshots/dataProducts/acceptAndInstall.png)

We clicked on the schema to store the name of the schema for later.

![get schema](./screenshots/dataProducts/getSchema.png)

We copied the schema from there.

![copy schema](./screenshots/dataProducts/copySchema.png)

==> The Data Product *Customer* had now been shared from SAP Business Data Cloud to SAP HANA Cloud and installed. 

#### 3.2.1.3 Making the shared Data Product available for consumption in Calculation Views

Next, we granted the required privileges so that you can create calculation views on the objects of the shared Data Product. To do so we made use of the role *\_SYS\_DI#BROKER_CG.\_SYS\_DI\_OO\_DEFAULTS* which is assigned to every object owner of HDI containers that are created in SAP Business Application Studio and role *BROKER\_USER.RT\_DEFAULTS* which is assigned to the application runtime users ending in *_RT* that are created during usage of SAP Business Application Studio.

Object owners require database privilege *SELECT WITH GRANT* on objects that should be used in calculation views. For runtime users privilege *SELECT* is sufficient.

With this in mind, 
- We created a role *DP\_CUSTOMER\_OO* for the assignment to the object owners of HDI containers created via SAP Business Application Studio
- Assigned the SELECT WITH GRANT privileges on the schema that we had copied above to the role *DP\_CUSTOMER\_OO*
- Finally granted role *DP\_CUSTOMER\_OO* to the default role *"\_SYS\_DI#BROKER\_CG"."\_SYS\_DI\_OO\_DEFAULTS"* of the object owners:

```SQL
create role DP_CUSTOMER_OO;
grant select on schema 	"_SAP_DATAPRODUCT_sap_s4com_dataProduct_Customer_v1_021ca5b5-1d3e-48c3-8b7b-eb8f2c7ba5ff" to DP_CUSTOMER_OO WITH GRANT OPTION;
grant DP_CUSTOMER_OO to "_SYS_DI#BROKER_CG"."_SYS_DI_OO_DEFAULTS";
```

We did the same with role *DP\_CUSTOMER\_RT* for the application users:

```SQL
create role DP_CUSTOMER_RT;
grant select on schema "_SAP_DATAPRODUCT_sap_s4com_dataProduct_Customer_v1_021ca5b5-1d3e-48c3-8b7b-eb8f2c7ba5ff" to DP_CUSTOMER_RT;
grant DP_CUSTOMER_RT to "BROKER_USER"."RT_DEFAULTS";
```

Having done these preparation steps for you, you can now create calculation views on the virtual tables that belong to the Data Product *Customer*.

It's now your turn again to build a calculation view of Data Category *Dimension* in SAP Business Application Studio on table *Customer* of the Data Product *Customer* and to join the dimension calculation view in star-join *s_products*. The goal is to enrich the transactional data with customer information.

### 3.2.2 Building Calculation Views on top of the shared Data Product *Customer*

- Switch back to SAP Business Application Studio

*Create calculation view "D_customer" of Data Category "Dimension" that is based on table "Customer" from Data Product "Customer"*

- Create Calculation View *D_customer* of type *Dimension*:

    - Right-click on folder *src* and choose *New File*:

        ![create new file](./screenshots/createSynonyms/newFile.png)

    - Enter *D_customer.hdbcalculationview* as filename and press *ENTER* to accept the name:

        ![create calculation view](./screenshots/dataProducts/createView.png)

    - Select *Dimension* (1.) and press *Create* (2.):

        ![create dimension view](./screenshots/dataProducts/chooseDimension.png)

==> The calculation view *D_customer* is created as a Dimension View.

- Click on node *Projection* so that the +-icon of *Add Data Source* becomes available (1.) and click on the icon (2.)

![add data source](./screenshots/dataProducts/addDataSource.png)

> **Note:** At the beginning of the session, synonyms were created in bulk. Since only
> one synonym is needed here, we will create it on the fly during modeling.

- Unselect *Show only local objects*...


- Deselect *Show only local objects* so that also objects outside of your HDI container become available for selecting (1.)
- Enter *customer* to search for the customer tables (2.)
- Choose the table that ends in *.Customer* (3.)
- Press *Create Synonym* (4.)

![select table](./screenshots/dataProducts/selectTable.png)

- In the next screen keep all the defaults and press *Finish*

![finish](./screenshots/dataProducts/finish.png)

==> A synonym file has been generated in the *EXPLORER* window on the left side and deployed. The synonym is added into your projection.

*Map all columns to the output*

- Double-click on node *Projection* to open the details (1.)
- Double-click on the data source to add all columns to the output (2.)

![map all columns to output](./screenshots/dataProducts/addAllColumnsToOutput.png)

- Deploy the calculation view with the rocket icon. Alternatively deploy it by pressing *CTRL* and *d*

![deploy](./screenshots/calculationview/dimension/deploy.png)

==> You have created and successfully deployed the dimension calculation view *D_customer*.

### 3.2.3 Integrating the Data Product *Customer* into an existing Star Join

*Join dimension view "D_customer" in the star-join of "s_products" on column "customerID"*

- Double-click on file *s_products* on the left *EXPLORER* window to open the calculation view in the editor

==> A dialog box informs you that the calculation view is shared to SAP Business Data Cloud and therefore changes to the current calculation view might break the models in SAP Datasphere that are based on the current calculation view. We will only enhance the calculation view with an additional dimension and thus not break the semantic onboarded model.

- Click on *OK* to proceed

![accept warning](./screenshots/dataProducts/warning.png)

- Click on node *Star Join* so that the + icon *Add Data Source* becomes available (1.) and click on the icon (2.)

![add dimension](./screenshots/dataProducts/addDimension.png)

- Search for dimension calculation views that contain the string *customer* by entering *customer* as the search string (1.)
- Select the calculation view *D_customer* (2.)
- Press *Finish* (3.)

![select dimension](./screenshots/dataProducts/selectDimension.png)

==> The dimension view *D_customer* has been added to node *Star Join*.

- Double-click on node *Star Join* to open the details (1.)
- Choose tab *Join Definition* (2.)
- If you don't see the newly added view *D_customer* click into the area with the sources (3.) and use the mouse wheel or the zoom buttons (4.) to zoom out until you see data source *D_customer*
- Click on column *customerID* of data source *Projection_1* (5.) and, while holding the mouse button down, move over to data source *D_customer* and release the mouse button on column *Customer* (6.) to define the join fields

![define join](./screenshots/dataProducts/defineJoin.png)

==> The join is now defined on column *customerID* of data source *Projection_1* and column *Customer* of data source *D_customer*. To avoid redundancies in the output, the column from the transactional data source *Projection_1* is removed from the output. A dialog informs about this removal. 

- Accept the removal by pressing button *Yes*

![remove](./screenshots/dataProducts/remove.png)

*Define the join details according to the fact-dimension context*

While multiple entries with the same customer ID typically appear in transactional data sources, the joined customer dimension has unique customer IDs. To reflect this:

- Click on the connecting line between *customerID* and *Customer* so that the columns are highlighted (1.)
- Under *PROPERTIES*: 
    - Select *Left Outer* as Join Type (2.) 
    - Select *n..1* as Cardinality (3.)

![define join details](./screenshots/dataProducts/defineJoinDetails.png)

- Deploy the calculation view with the rocket icon. Alternatively deploy it by pressing *CTRL* and *d*

![deploy](./screenshots/calculationview/dimension/deploy.png)

### 3.2.4 Run a Data Preview to see data from the shared Data Product *Customer*

*Run a Data Preview to see the enrichment with customer information*

- Right-click on node *Star Join* (1.) and select option *SQL Data Preview* (2.) to generate an SQL statement that requests all available columns

![SQL data preview](./screenshots/dataProducts/SQLDataPreview.png)

- Click on the *Run* button to execute the generated SQL statement and verify that now data from the Data Product *Customer* are available in columns such as *Country* or *CustomerAccountGroup*.

![run SQL data preview](./screenshots/dataProducts/run.png)

==> You have shared the Data Product *Customer* from SAP Business Data Cloud to SAP HANA Cloud, and integrated this customer information into your existing calculation view to enrich it with customer data.

With this you have now learned to create calculation views, include them in Data Exploration tools (exercise 2.0), share them into SAP Datasphere via Semantic Onboarding (exercise 3.1), and to use calculation views on shared Data Products (exercise 3.2). This knowledge will help you to achieve your goals with SAP HANA Cloud in SAP Business Data Cloud.