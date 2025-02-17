---
auto_validation: true
title: Define CDS-Based Travel Data Model
description: Define CDS-based data model and create projection view.
primary_tag: products>sap-cloud-platform--abap-environment
tags: [  tutorial>beginner, topic>abap-development, products>sap-cloud-platform ]
time: 20
author_name: Merve Temel
author_profile: https://github.com/mervey45
---

## Prerequisites  
- You have created an SAP Cloud Platform ABAP environment trial user or
- You have created a developer user in an SAP Cloud Platform ABAP Environment system.
- You have downloaded Eclipse Photon or Oxygen and installed ABAP Development Tools (ADT). See <https://tools.hana.ondemand.com/#abap>.

## Details
### You will learn  
  - How to create CDS based data model
  - How to create projection view
  - How to create service definition
  - How to create service binding

In this tutorial, wherever XXX appears, use a number (e.g. 000).

--- 

[ACCORDION-BEGIN [Step 1: ](Define CDS-based travel data model)]
  1. Right-click on your package `ZTRAVEL_APP_XXX`, select **New** > **Other ABAP Repository Object**.

      ![Define CDS based travel data model](model.png)

  2. Search for data definition, select it and click **Next >**.

      ![Define CDS based travel data model](model2.png)

  3. Create a data definition:

     - Name: `ZI_TRAVEL_M_XXX`
     - Description: `Data model for travel`

     Click **Next >**.

      ![Define CDS based travel data model](model3.png)

  4. Click **Finish**.

      ![Define CDS based travel data model](model4.png)

  5. Define root view for `ZI_TRAVEL_M_XXX` and database table as source.

    ```ABAP
    define root view ZI_TRAVEL_M_XXX as select from ztravel_xxx            
    ```

  6. Replace your code with following:

    ```ABAP
    @AbapCatalog.sqlViewName: 'ZVI_TRAVEL_M_XXX'
    @AbapCatalog.compiler.compareFilter: true
    @AbapCatalog.preserveKey: true
    @AccessControl.authorizationCheck: #CHECK
    @EndUserText.label: 'Travel data - XXX'
    define root view ZI_TRAVEL_M_XXX

     as select from ztravel_xxx as Travel

     /* Associations */
     association [0..1] to /DMO/I_Agency   as _Agency   on $projection.agency_id = _Agency.AgencyID
     association [0..1] to /DMO/I_Customer as _Customer on $projection.customer_id = _Customer.CustomerID
     association [0..1] to I_Currency      as _Currency on $projection.currency_code = _Currency.Currency
     {
      key travel_id,
         agency_id,
         customer_id,
         begin_date,
         end_date,
         @Semantics.amount.currencyCode: 'currency_code'
         booking_fee,
         @Semantics.amount.currencyCode: 'currency_code'
         total_price,
         @Semantics.currencyCode: true
         currency_code,
         overall_status,
         description,

    /*-- Admin data --*/
         @Semantics.user.createdBy: true
         created_by,
         @Semantics.systemDateTime.createdAt: true
         created_at,
         @Semantics.user.lastChangedBy: true
         last_changed_by,
         @Semantics.systemDateTime.lastChangedAt: true
         last_changed_at,

         /* Public associations */
         _Agency,
         _Customer,
         _Currency
    }        
    ```

  7. Save and activate.

     Your CDS view for travel booking is defined now. You can use and manipulate data that is persisted in your database.

[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 2: ](Create projection view for travel)]
  1. Right-click on your package `ZTRAVEL_APP_XXX`, select **New** > **Other ABAP Repository Object**.

      ![Create projection view for travel](projection.png)

  2. Search for data definition, select it and click **Next >**.

      ![Create projection view for travel](projection2.png)

  3. Create a data definition:

     - Name: `ZC_TRAVEL_M_XXX`
     - Description: `Projection view for travel`

     Click **Next >**.

      ![Create projection view for travel](projection3.png)

  4. Click **Finish**.

      ![Create projection view for travel](projection4.png)

  5. Define root view entity for `ZC_TRAVEL_M_XXX`.

    ```ABAP
    define root view entity ZC_TRAVEL_M_XXX as projection on ZI_Travel_M_XXX        
    ```

  6. Replace your code with following:

    ```ABAP
    @EndUserText.label: 'Travel projection view - Processor'
    @AccessControl.authorizationCheck: #NOT_REQUIRED

    @UI: {
     headerInfo: { typeName: 'Travel', typeNamePlural: 'Travels', title: { type: #STANDARD, value: 'TravelID' } } }

    @Search.searchable: true

    define root view entity ZC_TRAVEL_M_XXX as projection on ZI_TRAVEL_M_XXX {
     @UI.facet: [ { id:              'Travel',
                    purpose:         #STANDARD,
                    type:            #IDENTIFICATION_REFERENCE,
                    label:           'Travel',
                    position:        10 } ]

     @UI: {
         lineItem:       [ { position: 10, importance: #HIGH } ],
         identification: [ { position: 10, label: 'Travel ID [1,...,99999999]' } ] }
     @Search.defaultSearchElement: true
     key travel_id          as TravelID,

     @UI: {
         lineItem:       [ { position: 20, importance: #HIGH } ],
         identification: [ { position: 20 } ],
         selectionField: [ { position: 20 } ] }
     @Consumption.valueHelpDefinition: [{ entity : {name: '/DMO/I_Agency', element: 'AgencyID'  } }]

     @ObjectModel.text.element: ['AgencyName'] ----meaning?
     @Search.defaultSearchElement: true
     agency_id          as AgencyID, _Agency.Name       as AgencyName,

     @UI: {
         lineItem:       [ { position: 30, importance: #HIGH } ],
         identification: [ { position: 30 } ],
         selectionField: [ { position: 30 } ] }
     @Consumption.valueHelpDefinition: [{ entity : {name: '/DMO/I_Customer', element: 'CustomerID'  } }]

     @ObjectModel.text.element: ['CustomerName']
     @Search.defaultSearchElement: true
     customer_id        as CustomerID,

     @UI.hidden: true
     _Customer.LastName as CustomerName,

     @UI: {
         lineItem:       [ { position: 40, importance: #MEDIUM } ],
         identification: [ { position: 40 } ] }
     begin_date         as BeginDate,

     @UI: {
         lineItem:       [ { position: 41, importance: #MEDIUM } ],
         identification: [ { position: 41 } ] }
     end_date           as EndDate,

     @UI: {
         lineItem:       [ { position: 50, importance: #MEDIUM } ],
         identification: [ { position: 50, label: 'Total Price' } ] }
     @Semantics.amount.currencyCode: 'CurrencyCode'
     total_price        as TotalPrice,

     @Consumption.valueHelpDefinition: [{entity: {name: 'I_Currency', element: 'Currency' }}]
     currency_code      as CurrencyCode,

     @UI: {
           lineItem:       [ { position: 60, importance: #HIGH },
                             { type: #FOR_ACTION, dataAction: 'acceptTravel', label: 'Accept Travel' } ],
         identification: [ { position: 60, label: 'Status [O(Open)|A(Accepted)|X(Canceled)]' } ]  }
     overall_status     as TravelStatus,

     @UI.identification: [ { position: 70, label: 'Remarks' } ]
     description as Description,

     @UI.hidden: true
     last_changed_at    as LastChangedAt

     }        
    ```

  7. Save and activate.

     The **projection** is created and includes UI annotations. The projection is the subset of the fields of the travel data model, which are relevant for the travel booking application.


[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 3: ](Create service definition)]
  1. Right-click on your data definition `ZC_TRAVEL_M_XXX` and select **New Service Definition**.

      ![Create service definition](definition.png)

  2. Create a new service definition:

     - Name: `ZUI_C_TRAVEL_M_XXX`
     - Description: `Service definition for travel`

     Click **Next >**.

      ![Create service definition](definition2.png)

  3. Click **Finish**.

      ![Create service definition](definition3.png)

  4. Replace your code with following:

    ```ABAP
    @EndUserText.label: 'Service Defintion for ZC_Travel_M_XXX'
    define service ZUI_C_TRAVEL_M_XXX
    {
      expose ZC_TRAVEL_M_XXX as TravelProcessor;
      expose /DMO/I_Customer as Passenger;
      expose /DMO/I_Agency as TravelAgency;
      expose /DMO/I_Airport as Airport;
      expose I_Currency as Currency;
      expose I_Country as Country;
    }   
    ```

  5. Save and activate.

     With the **service definition** you are able to define which data is exposed as a business service in your travel booking application.

[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 4: ](Create service binding)]
  1. Right-click on your service definition `ZUI_C_TRAVEL_M_XXX` and select **New Service Binding**.

      ![Create service binding](binding.png)

  2. Create a new service binding:

     - Name: `ZUI_C_TRAVEL_M_XXX`
     - Description: `Service binding for travel`
     - Binding Type: `ODATA V2 - UI`

     Click **Next >**.

      ![Create service binding](binding2.png)

  3. Click **Finish**.

      ![Create service binding](binding3.png)

  4. Activate your service binding.

      ![Create service binding](binding4.png)

     Now your service binding is created and you are able to see your service with its entities and associations.

     The **service binding** allows you to bind the service definition to an ODATA protocol. Therefore you are able to see the travel booking application on the UI.

  5. Double-click on `TravelProcessor` to see the application on the UI.

      ![Create service binding](binding5.png)

  6. Logon to your ABAP system.

      ![Create service binding](binding6.png)

  7. Click **GO** to see your result.

     Your columns are created. The annotation ensures that all columns are already selected. As you can see, buttons like, create and update are missing. Therefore you need to define your behavior definition.

      ![Create service binding](binding7.png)

[DONE]
[ACCORDION-END]


[ACCORDION-BEGIN [Step 5: ](Test yourself)]

[VALIDATE_1]
[ACCORDION-END]
---
