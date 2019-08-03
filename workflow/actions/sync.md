Workflow Rules to sync data
===========================

## Create and update food
This is a rule to add a new Food record whenever a new Account gets added or edited.

### When condition
The when condition is set on create or edit, while theoreticaly edit is not needed we keep this as a safe guard for any issues.
![when condition: create or edit with repeat](./images/updatefood/when.png)

### Condition 1
We apply the rule to all accounts
This runs a function called `updateFood` which receives an Account ID as `accountID`.
```deluge
user = zoho.crm.getRecordById("Accounts",accountID);
contacts = zoho.crm.getRelatedRecords("Contacts","Accounts",klantID,1,100); // children are stored in contacts

numAdult = 1;
numChild = 0;
birthData = "";
for each  contact in contacts
{
	birthdate = contact.get("Date_of_Birth");
	if(birthdate != null)
	{
		birthData = birthData + birthdate + "\n";
		age = ((zoho.currentdate.toDate() - birthdate.toDate()) / (1000 * 3600 * 24 * 365)).floor();
		if(age < 12) // food amount whise
		{
			numChild = numChild + 1;
		}
		else
		{
			numAdult = numAdult + 1;
		}
	}
}

records = zoho.crm.searchRecords("Food","(Client:equals:" + accountID + ")",1,100);
exists = false;
for each record in records
{
	userID = record.get("User").get("id");
	if(userID.toString() == accountID.toString())
	{
		exists = true;

		// This copies the personal info fields
		// You need to add a line for every field
		record.put("Name",user.get("Name"));
		record.put("Number_of_Adults",numAdult);
		record.put("Number_of_Children",numChild);

		zoho.crm.updateRecord("Food",record.get("id"),record);
	}
}

if (!exists)
{
	mp = Map();

	userMP = Map();
	userMP.put("id",accountID);

    // This copies the personal info fields
    // You need to add a line for every field
	mp.put("Name",gebruiker.get("Name")); 

	mp.put("Gebruiker",userMP);

	created = zoho.crm.createRecord("Food",mp);
	info mp; // handy debug lines
	info created;
}
```
