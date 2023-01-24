#**Chapter 3 - Extending the Domain Model (incl. Changes to the front-end)**

Let's customize the model so we can update the information we need to properly track the status and location of members.

In this section, you learn how to:
- ✅ Extend an Existing Entity
- ✅ Create a Reference List
- ✅ Create a Migration Class
- ✅ Updating the Existing Table

##**Prerequisites**
In order to complete the tutorial successfully, you will need to complete the previous section on **Configuring Your First View** and ensure the following items are set up.
- A Test Environment with Admin Access
- [Visual Studio Code](https://code.visualstudio.com/)
- [Visual Studio](https://visualstudio.microsoft.com/)

#**Backend**

In the next few sections we are going to be diving into a bit of code. You can open up the **backend** folder that you downloaded from the starter project folder. In our case it is called **Boxfusion.Membership.sln**.

![image.png](/.attachments/image-57f65379-f1e7-447b-bc2f-161b0c3ad89a.png)

We will be working in the **Module > Boxfusion.Membership.Common.Domain > Domain** folder to add our new Member code and logic.

##**1. Adding a Table Prefix in the AssemblyInfo.cs**

In order to be able to reference our data from the database we will be adding a new table and fields using a code first approach.
1. Open the **AssemblyInfo.cs** file in **Module > Boxfusion.Membership.Common.Domain > Properties** folder.
1. In the AssemblyInfo.cs, you will find an attribute tag, e.g. **[assembly: TablePrefix("Membership_")]** - shorten the “Membership”  to “Mem”, as an easier convention for naming your table prefixes. It should look like this when complete: **[assembly: TablePrefix("Mem_")]**.

The table prefix above is what we will use to identify the tables belonging to the Membership Management application we are building.


##**2. Create a new class**

1. Navigate to **Module > Boxfusion.Membership.Common.Domain > Domain**
1. Right click on the **Domain** folder, **Add > Class**
1. Give your class the name of: **Member.cs**, and click on **Add**- This is where we will be adding properties we want as an addition to the **Person** table.

This is how your class should be constructed: 

```
using Shesha.Domain.Attributes;
using Shesha.Domain;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.ComponentModel.DataAnnotations.Schema;
 
namespace Boxfusion.Membership.Common.Domain.Domain
{
 
        // <summary>
        /// A person within the application that is a Member
        /// </summary>
        [Entity(TypeShortAlias = "Mem.Member")]
        public class Member : Person
        {
            /// <summary>
            /// The membership number for the Member
            /// </summary>
            public virtual string MembershipNumber { get; set; }
            /// <summary>
            /// The Members residential address
            /// </summary>
            public virtual string ResidentialAddress { get; set; }
            /// <summary>
            /// The region that the Member belongs to
            /// </summary>
            public virtual Area Region { get; set; }
            /// <summary>
            /// The branch that the Member belongs to
            /// </summary>
            public virtual Area Branch { get; set; }
            /// <summary>
            /// The date when the Members membership started
            /// </summary>
            public virtual DateTime MembershipStartDate { get; set; }
            /// <summary>
            /// The date when the Members membership ended
            /// </summary>
            public virtual DateTime MembershipEndDate { get; set; }


            /// <summary>
            /// Identification document for the Member
            /// </summary>
            [NotMapped]
            public virtual StoredFile IdDocument { get; set; }
        }
}

```

##**3. Create a Migration Class**
We will now create a migration class in order to update the ‘Person’ table in our database to have properties defined in the ‘Member’ domain.
1. Navigate to **Module > Boxfusion.Membership.Common.Domain > Migrations**
1. Right click on the **Migrations** folder, **Add > Class**
1. Create a new migration class with a file name following this format: **M[YEAR][MONTH][DAY][HOUR][MINUTE][SECONDS]**.cs e.g. M20220805085300.cs for 5 August 2022 08:53:00.
1. Add the below code:


```
using FluentMigrator;
using Shesha.FluentMigrator;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
 
// <summary>
/// Adding the Members table
/// </summary>

[Migration(20221208132400)]
public class M20221208132400 : Migration
{
    /// <summary>
    /// Code to execute when executing the migrations
    /// </summary>
    public override void Up()
    {
        Alter.Table("Core_Persons")
            .AddColumn("Mem_MembershipNumber").AsString().Nullable()
            .AddColumn("Mem_ResidentialAddress").AsString().Nullable()
            .AddForeignKeyColumn("Mem_RegionId", "Core_Areas").Nullable()
            .AddForeignKeyColumn("Mem_BranchId", "Core_Areas").Nullable()
            .AddColumn("Mem_MembershipStartDate").AsDateTime().Nullable()
            .AddColumn("Mem_MembershipEndDate").AsDateTime().Nullable();
    }
    /// <summary>
    /// Code to execute when rolling back the migration
    /// </summary>
    public override void Down()
    {
        throw new NotImplementedException();
    }
}


```

5. You can run your application by going to the menu and selecting **Debug > Start Debugging** or by clicking **F5**
6. Go to your **Microsoft SQL Server Management Studio** and query the **Person** table in your database, where you will see that its properties have been extended with the properties from the **Members** domain. 

<IMG  src="https://lh5.googleusercontent.com/hZhoc6prnOiNSoTGkc2fTLxEHeyXA1fR7MzX_a4KMsd6gdZEsgL7mdJ7qG8CHE6AP0Wd2WiJusNDZPYZPKLW1IelrOIlomMfcY5O9ot53JDjsH8Tqjn1djoYwDVsw8I0l2CwdpZHtX37qlw31aRwnE2SnGCToomW-NjjEXO5jJ-HW7I3cXMlA-ZPY2DU1A"  width="624"  height="485" style="margin-left:0px;margin-top:0px"/>
<br>
<br>

You can check out [Fluent Migrator](https://fluentmigrator.github.io/index.html) for more options about database migrations.

##**4. Create a Reference List**
1. Create a folder called **Enums** in **Module > Boxfusion.Membership.Common.Domain**. 
1. Right click on the **Enums** folder, **Add > Class**.
1. Give your class the name of: **RefListMembershipStatuses.cs**, and click on **Add**.
1. Add the below code:


```
using Shesha.Domain.Attributes;
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
 
namespace Boxfusion.Membership.Common.Domain.Enums
{
    /// <summary>
    /// Statuses for a Members Membership
    /// </summary>
    [ReferenceList("Mem", "MembershipStatuses")]
    public enum RefListMembershipStatuses : long
    {
        /// <summary>
        /// Membership status is still being processed
        /// </summary>
        [Description("In Progress")]
        InProgress = 1,
        /// <summary>
        /// Membership status is active
        /// </summary>
        [Description("Active")]
        Active = 2,
        /// <summary>
        /// Membership status is cancelled
        /// </summary>
        [Description("Cancelled")]
        Cancelled = 3
    }
}
```


##**5. Updating the Existing Table**
Now that we have created the **RefListMembershipStatuses** you can go and add the following code below to your **Member.cs** entity:

This is how the file should look like in the end: 

```
using Shesha.Domain.Attributes;
using Shesha.Domain;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.ComponentModel.DataAnnotations.Schema;
using Boxfusion.Membership.Common.Domain.Enums;
 
namespace Boxfusion.Membership.Common.Domain.Domain
{
 
        // <summary>
        /// A person within the application that is a Member
        /// </summary>
        [Entity(TypeShortAlias = "Mem.Member")]
        public class Member : Person
        {
            /// <summary>
            /// The membership number for the Member
            /// </summary>
            public virtual string MembershipNumber { get; set; }
            /// <summary>
            /// The Members residential address
            /// </summary>
            public virtual string ResidentialAddress { get; set; }
            /// <summary>
            /// The region that the Member belongs to
            /// </summary>
            public virtual Area Region { get; set; }
            /// <summary>
            /// The branch that the Member belongs to
            /// </summary>
            public virtual Area Branch { get; set; }
            /// <summary>
            /// The date when the Members membership started
            /// </summary>
            public virtual DateTime MembershipStartDate { get; set; }
            /// <summary>
            /// The date when the Members membership ended
            /// </summary>
            public virtual DateTime MembershipEndDate { get; set; }
            /// <summary>
            /// Identification document for the Member
            /// </summary>
            [NotMapped]
            public virtual StoredFile IdDocument { get; set; }
            /// <summary>
            /// The status of the membership
            /// </summary>
            public virtual RefListMembershipStatuses? MembershipStatus { get; set; }
    }
    
}

```

##**6. Create a Migration Class**
We will now create a migration class in order to update the **Person** table in our database to have the new reference list you have just added to the **Member** entity.


```
using FluentMigrator;
using Shesha.FluentMigrator;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
 
namespace Boxfusion.Membership.Common.Domain.Migrations
{
    [Migration(20221213114350)]
    public class M20221213114350: Migration
    {
        /// <summary>
        /// Code to execute when executing the migrations
        /// </summary>
        public override void Up()
        {
            Alter.Table("Core_Persons")
               .AddColumn("Mem_MembershipStatusLkp").AsInt64().Nullable();
        }
        /// <summary>
        /// Code to execute when rolling back the migration
        /// </summary>
        public override void Down()
        {
            throw new NotImplementedException();
        }
    }
}
```

##**7. Run and check database**
You can run your application in order to update your database. Your new updated column should be now added to your table.


<IMG  src="https://lh3.googleusercontent.com/LrlASlhJvGSciOwm5QUnQBEEFygc-uPjalPOFUntu_W1ROinT5Gn-uJif6SONF3LPgtfx6j7bslRVL3ORSErFV_MVKRleK54apK0UgzU0gTtKAhRWoN0oyQn0bsLP-C9xQ2EJTtH3Gy2JyPqspTM6SDxBQcFmrz5394LYjDCQgl-t-0ZYnCTDo2hsaABRA"  width="624"  height="656" style="margin-left:0px;margin-top:0px"/>

<br>
<br>

#**Configurations**
##**Updating the Binding Model for Our Views**

Now that we have fully extended our domain model, it is time to go back and update our views so that we can reference our newly created fields in the **Member** entity.

This can be done by updating the **Model Type** property in all our views from **Shesha.Domain.Person (Shesha.Core.Person)** to **Boxfusion.Membership.Common.Domain.Domain.Member (Mem.Member)**, changing CRUD endpoints to point to the relevant model type, and adding the following fields:

- **MembershipNumber** -  Textfield: string
- **ResidentialAddress** -  Textfield: string
- **MembershipStatus** - RadioButton: RefListMembershipStatuses
- **MembershipStartDate** - Datefield: DateTime
- **MembershipEndDate** - Datefield: DateTime

You can go reference the **Configuring Your First View** chapter if you get stuck with the next steps.

##**1. Member Create View**
![image.png](/.attachments/image-aac820b3-5ed9-444a-8f51-316589c56bec.png)

<br>

![image.png](/.attachments/image-658991a1-a295-4ef2-9168-c8a249cd90f1.png)

![image.png](/.attachments/image-fd1860b9-d2d1-498f-ab6b-874c80477b2f.png)


##**2. Member Details View**

![image.png](/.attachments/image-1a064cc8-964b-4c62-9deb-ca18fe90a841.png)

<br>

![image.png](/.attachments/image-d04b91c5-3edc-451d-be9b-2066c9a472ac.png)


![image.png](/.attachments/image-53dcbab3-fd09-4526-ba50-42ba341b92b7.png)

##**3. Member Table**

![image.png](/.attachments/image-79a7c64c-95b5-40d9-9c10-30bc82f7eb8f.png)

- Click on the **datatableContext** component handler, and search for the **Member** entity on the **Entity Type** option in the **Properties** sidebar.

![image.png](/.attachments/image-d106f9c4-0472-4be8-9635-36784796f153.png)



- Click on the **datatable** component handler,and click on the **Customize Columns** option in the **Properties** sidebar to add the relevant columns.

![image.png](/.attachments/image-4213406f-909e-471f-b869-d399b22c8593.png)


<br>

![image.png](/.attachments/image-71f95e1e-78b0-4e26-8deb-520852bf76a7.png)

<br>

![image.png](/.attachments/image-b4465fe5-6e27-42cb-81bd-0d18c7a79e77.png)
