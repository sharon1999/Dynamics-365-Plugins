# GET Values
```cs
public void GetFields()
        {
            string fetchXmlString = @"<fetch version='1.0' output-format='xml-platform' mapping='logical' distinct='false' count='5'>
                                          <entity name='lead'>
                                            <attribute name='companyname' />
                                            <attribute name='leadid' />
                                            <attribute name='businesstype' />
                                            <attribute name='decisionmaker' />
                                            <attribute name='numberofemployees' />
                                            <attribute name='address1_longitude' />
                                            <attribute name='exchangerate' />
                                            <attribute name='revenue' />
                                            <attribute name='description' />
                                            <attribute name='rejectiondate' />
                                            <attribute name='parentaccountid' />
                                            <order attribute='companyname' descending='false' />
                                            <filter type='and'>
                                                <condition attribute='parentaccountid' operator='not-null'/>
                                            </filter>
                                          </entity>
                                        </fetch>";
            EntityCollection entColl = crmService.RetrieveMultiple(new FetchExpression(fetchXmlString));
            if (entColl != null && entColl.Entities != null && entColl.Entities.Count > 0)
            {
                Entity entLead = entColl.Entities[0];
 
                //-- Option Set
                if (entLead.Contains("businesstype") && entLead["businesstype"] != null)
                {
                    OptionSetValue businesstype = (OptionSetValue) entLead["businesstype"]; 
                    int selectedOptionValue = businesstype.Value;
                }
 
                //--Two option field
                if (entLead.Contains("decisionmaker") && entLead["decisionmaker"] != null)
                {
                    bool decisionmaker = (bool) entLead["decisionmaker"]; 
                }
 
                //-- Whole Number field
                if (entLead.Contains("numberofemployees") && entLead["numberofemployees"] != null)
                {
                    int numberofemployees = (int) entLead["numberofemployees"]; 
                }
 
                //-- Floating Point Number field
                if (entLead.Contains("address1_longitude") && entLead["address1_longitude"] != null)
                {
                    double address1_longitude = (double) entLead["address1_longitude"]; 
                }
 
                //--Decimal Number field
                if (entLead.Contains("exchangerate") && entLead["exchangerate"] != null)
                {
                    decimal exchangerate = (decimal)entLead["exchangerate"]; 
                }
 
                //--Currency field
                if (entLead.Contains("revenue") && entLead["revenue"] != null)
                {
                    Money revenue = (Money) entLead["revenue"]; 
                    decimal revenue_value = revenue.Value;
                }
 
                //--Date and Time field
                if (entLead.Contains("rejectiondate") && entLead["rejectiondate"] != null)
                {
                    DateTime rejectiondate = (DateTime) entLead["rejectiondate"]; 
                }
 
                //-- Lookup field
                if (entLead.Contains("parentaccountid") && entLead["parentaccountid"] != null)
                {
                    EntityReference parentaccountid = (EntityReference) entLead["parentaccountid"]; 
                    Guid id = parentaccountid.Id;
                    string logicalName = parentaccountid.LogicalName;
                    string name = parentaccountid.Name;
                }
            }
        }
```

# SET Values
```cs
public void SetFields()
        {
            Entity entLead = new Entity("lead");
            entLead["leadid"] = new Guid("21cb74e2-6254-e811-80dc-005056be7607");
 
            //-- Option Set
            entLead["businesstype"] = new OptionSetValue(0); //-- Individual = 0, Business = 1
 
            //--Two option field
            entLead["decisionmaker"] = true;
 
            //-- Whole Number field
            entLead["numberofemployees"] = 10;
 
            //-- Floating Point Number field
            entLead["address1_longitude"] = 110.123;
 
            //--Decimal Number field
            entLead["exchangerate"] = 11.21;
 
            //--Currency field
            Money revenue = new Money();
            revenue.Value = new decimal(2351.1123);
            entLead["revenue"] = revenue;
 
            //--Date and Time field
            entLead["rejectiondate"] = DateTime.Now;
 
            //-- Lookup field
            EntityReference parentaccountid = new EntityReference("account",new Guid("5DFCDD2D-FCE8-E311-940C-005056BD2B49"));
            entLead["parentaccountid"] = parentaccountid;
            // or this can be written as 
            entLead["parentaccountid"] = account.toEntityReference();
            
 
            crmService.Update(entLead);
        }
```
