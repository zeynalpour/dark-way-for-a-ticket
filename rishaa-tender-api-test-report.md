# Rishaa tender submission API test report

Date: 2026-11-08

Documentation: https://rishaa.com/swagger/docs/v1

## Scope and result

Five submission variants were tested. Every request returned HTTP 500 with the same Entity Framework validation exception. No successful tender or draft creation was confirmed. The draft list was inspected after the final attempt; the existing draft titled `تست` remained first, and no new test draft was observed on that page.

The first prepared request was executed by the user; its response was inspected in Swagger. Subsequent requests were executed through Swagger during this session.

## Common request settings

```http
POST https://rishaa.com/api/TenderCompletion/SendData
Authorization: Bearer <REDACTED>
Content-Type: application/json
Accept: application/json
```

The API credential is intentionally excluded. Each JSON body below is a complete request body, not a patch. Response bodies below retain the error fields; the repetitive `StackTrace` field is omitted and its relevant location is recorded separately.

## Test 1: Initial submission: zero IDs and organization ID only

Server response timestamp (UTC): `2026-11-08T16:59:27Z`

### Request body

```json
{
  "TenderItemId": 0,
  "City": "سمنان",
  "Address": "تست — اتصال API ریشا",
  "desc": "تست — ثبت آزمایشی اتصال API؛ اطلاعات واقعی مناقصه نیست.",
  "DaysLeftToPay": 0,
  "FinancialSupplySourceID": 0,
  "Organization": {
    "Id": 32
  },
  "Audits": [],
  "Products": [
    {
      "ProductTitle": "تست — کالای آزمایشی اتصال API",
      "UnitType": "عدد",
      "Count": 1,
      "UnitPrice": 0,
      "Transport": 0,
      "PurchasePrice": 0,
      "OtherPrice": 0,
      "Insurance": 0,
      "Inspection": false,
      "Explain": "تست",
      "SalePrice": 0,
      "ProfitPercent": 0,
      "HaveSetup": false,
      "HaveInspection": false,
      "HaveUnloadingCost": false,
      "HaveProfit": false
    }
  ],
  "PrevTenders": []
}
```

### Output

HTTP status: **500 Internal Server Error**

```json
{
  "Message": "An error has occurred.",
  "ExceptionMessage": "Validation failed for one or more entities. See 'EntityValidationErrors' property for more details.",
  "ExceptionType": "System.Data.Entity.Validation.DbEntityValidationException"
}
```

Relevant stack location: `System.Data.Entity.Internal.InternalContext.SaveChanges()` → `TenderCompletionController.CompleteTenderProfile`, `TenderCompletionController.cs:line 129`.

## Test 2: Complete organization and explicit numeric/boolean values

Server response timestamp (UTC): `2026-11-08T17:13:37Z`

### Request body

```json
{
  "TenderItemId": 0,
  "City": "سمنان",
  "Address": "تست — نشانی آزمایشی اتصال API",
  "desc": "تست — مناقصه آزمایشی اتصال API ریشا؛ فاقد اعتبار عملیاتی",
  "DaysLeftToPay": 1,
  "FinancialSupplySourceID": 0,
  "Organization": {
    "Id": 32,
    "Title": "شرکت آب و فاضلاب شهری استان سمنان ",
    "Tel": "02331226000 ",
    "Address": "سمنان - بلوار بسیج - روبروی استانداری - کد پستی 3519863131",
    "City": "سمنان",
    "ActivityClassification": "آب و فاضلاب"
  },
  "Audits": [],
  "Products": [
    {
      "ProductTitle": "تست — کالای آزمایشی شماره ۲",
      "UnitType": "عدد",
      "Count": 1,
      "UnitPrice": 1000,
      "Transport": 0,
      "PurchasePrice": 1000,
      "OtherPrice": 0,
      "Insurance": 0,
      "Inspection": false,
      "Explain": "تست — داده ساختگی برای بررسی اعتبارسنجی API",
      "SalePrice": 1000,
      "ProfitPercent": 0,
      "HaveSetup": false,
      "HaveInspection": false,
      "HaveUnloadingCost": false,
      "HaveProfit": false
    }
  ],
  "PrevTenders": []
}
```

### Output

HTTP status: **500 Internal Server Error**

```json
{
  "Message": "An error has occurred.",
  "ExceptionMessage": "Validation failed for one or more entities. See 'EntityValidationErrors' property for more details.",
  "ExceptionType": "System.Data.Entity.Validation.DbEntityValidationException"
}
```

Relevant stack location: `System.Data.Entity.Internal.InternalContext.SaveChanges()` → `TenderCompletionController.CompleteTenderProfile`, `TenderCompletionController.cs:line 129`.

## Test 3: TenderItemId and FinancialSupplySourceID omitted

Server response timestamp (UTC): `2026-11-08T17:14:54Z`

### Request body

```json
{
  "City": "سمنان",
  "Address": "تست — نشانی آزمایشی اتصال API",
  "desc": "تست — مناقصه آزمایشی اتصال API ریشا؛ فاقد اعتبار عملیاتی",
  "DaysLeftToPay": 1,
  "Organization": {
    "Id": 32,
    "Title": "شرکت آب و فاضلاب شهری استان سمنان ",
    "Tel": "02331226000 ",
    "Address": "سمنان - بلوار بسیج - روبروی استانداری - کد پستی 3519863131",
    "City": "سمنان",
    "ActivityClassification": "آب و فاضلاب"
  },
  "Audits": [],
  "Products": [
    {
      "ProductTitle": "تست — کالای آزمایشی شماره ۲",
      "UnitType": "عدد",
      "Count": 1,
      "UnitPrice": 1000,
      "Transport": 0,
      "PurchasePrice": 1000,
      "OtherPrice": 0,
      "Insurance": 0,
      "Inspection": false,
      "Explain": "تست — داده ساختگی برای بررسی اعتبارسنجی API",
      "SalePrice": 1000,
      "ProfitPercent": 0,
      "HaveSetup": false,
      "HaveInspection": false,
      "HaveUnloadingCost": false,
      "HaveProfit": false
    }
  ],
  "PrevTenders": []
}
```

### Output

HTTP status: **500 Internal Server Error**

```json
{
  "Message": "An error has occurred.",
  "ExceptionMessage": "Validation failed for one or more entities. See 'EntityValidationErrors' property for more details.",
  "ExceptionType": "System.Data.Entity.Validation.DbEntityValidationException"
}
```

Relevant stack location: `System.Data.Entity.Internal.InternalContext.SaveChanges()` → `TenderCompletionController.CompleteTenderProfile`, `TenderCompletionController.cs:line 129`.

## Test 4: User-provided TenderItemId; funding source still omitted

Server response timestamp (UTC): `2026-11-08T17:16:42Z`

### Request body

```json
{
  "City": "سمنان",
  "Address": "تست — نشانی آزمایشی اتصال API",
  "desc": "تست — مناقصه آزمایشی اتصال API ریشا؛ فاقد اعتبار عملیاتی",
  "DaysLeftToPay": 1,
  "Organization": {
    "Id": 32,
    "Title": "شرکت آب و فاضلاب شهری استان سمنان ",
    "Tel": "02331226000 ",
    "Address": "سمنان - بلوار بسیج - روبروی استانداری - کد پستی 3519863131",
    "City": "سمنان",
    "ActivityClassification": "آب و فاضلاب"
  },
  "Audits": [],
  "Products": [
    {
      "ProductTitle": "تست — کالای آزمایشی شماره ۲",
      "UnitType": "عدد",
      "Count": 1,
      "UnitPrice": 1000,
      "Transport": 0,
      "PurchasePrice": 1000,
      "OtherPrice": 0,
      "Insurance": 0,
      "Inspection": false,
      "Explain": "تست — داده ساختگی برای بررسی اعتبارسنجی API",
      "SalePrice": 1000,
      "ProfitPercent": 0,
      "HaveSetup": false,
      "HaveInspection": false,
      "HaveUnloadingCost": false,
      "HaveProfit": false
    }
  ],
  "PrevTenders": [],
  "TenderItemId": 1105001469001057
}
```

### Output

HTTP status: **500 Internal Server Error**

```json
{
  "Message": "An error has occurred.",
  "ExceptionMessage": "Validation failed for one or more entities. See 'EntityValidationErrors' property for more details.",
  "ExceptionType": "System.Data.Entity.Validation.DbEntityValidationException"
}
```

Relevant stack location: `System.Data.Entity.Internal.InternalContext.SaveChanges()` → `TenderCompletionController.CompleteTenderProfile`, `TenderCompletionController.cs:line 129`.

## Test 5: Structure matching retrieved records: funding source 2 and nullable product fields

Server response timestamp was not captured in the report.

### Request body

```json
{
  "City": "سمنان",
  "Address": "تست — نشانی آزمایشی اتصال API",
  "desc": "تست — مناقصه آزمایشی اتصال API ریشا؛ فاقد اعتبار عملیاتی",
  "DaysLeftToPay": 30,
  "Organization": {
    "Id": 32,
    "Title": "شرکت آب و فاضلاب شهری استان سمنان ",
    "Tel": "02331226000 ",
    "Address": "سمنان - بلوار بسیج - روبروی استانداری - کد پستی 3519863131",
    "City": "سمنان",
    "ActivityClassification": "آب و فاضلاب"
  },
  "Audits": [],
  "Products": [
    {
      "ProductTitle": "تست — کالای آزمایشی شماره ۲",
      "UnitType": "عدد",
      "Count": 1,
      "UnitPrice": null,
      "Transport": null,
      "PurchasePrice": null,
      "OtherPrice": null,
      "Insurance": null,
      "Inspection": false,
      "Explain": "تست — داده ساختگی برای بررسی اعتبارسنجی API",
      "SalePrice": 1000,
      "ProfitPercent": 0,
      "HaveSetup": null,
      "HaveInspection": null,
      "HaveUnloadingCost": null,
      "HaveProfit": null
    }
  ],
  "PrevTenders": [],
  "TenderItemId": 0,
  "FinancialSupplySourceID": 2
}
```

### Output

HTTP status: **500 Internal Server Error**

```json
{
  "Message": "An error has occurred.",
  "ExceptionMessage": "Validation failed for one or more entities. See 'EntityValidationErrors' property for more details.",
  "ExceptionType": "System.Data.Entity.Validation.DbEntityValidationException",
  "StackTrace": "   at System.Data.Entity.Internal.InternalContext.SaveChanges()\r\n   at MeyarGostar.App.apiController.TenderCompletionController.<CompleteTenderProfile>d__5.MoveNext() in E:\\Project\\Meyar_new\\MeyarGostar.App\\apiController\\TenderCompletionController.cs:line 129\r\n--- End of stack trace from previous location where exception was thrown ---\r\n   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()\r\n   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)\r\n   at System.Threading.Tasks.TaskHelpersExtensions.<CastToObject>d__1`1.MoveNext()\r\n--- End of stack trace from previous location where exception was thrown ---\r\n   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()\r\n   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)\r\n   at System.Web.Http.Controllers.ApiControllerActionInvoker.<InvokeActionAsyncCore>d__1.MoveNext()\r\n--- End of stack trace from previous location where exception was thrown ---\r\n   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()\r\n   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)\r\n   at System.Web.Http.Controllers.ActionFilterResult.<ExecuteAsync>d__5.MoveNext()\r\n--- End of stack trace from previous location where exception was thrown ---\r\n   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()\r\n   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)\r\n   at System.Web.Http.Filters.AuthorizationFilterAttribute.<ExecuteAuthorizationFilterAsyncCore>d__3.MoveNext()\r\n--- End of stack trace from previous location where exception was thrown ---\r\n   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()\r\n   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)\r\n   at System.Web.Http.Dispatcher.HttpControllerDispatcher.<SendAsync>d__15.MoveNext()"
}
```

Relevant stack location: `System.Data.Entity.Internal.InternalContext.SaveChanges()` → `TenderCompletionController.CompleteTenderProfile`, `TenderCompletionController.cs:line 129`.

## Interpretation and remaining questions

- JSON numbers, booleans, omitted IDs, a supplied 64-bit ID, complete organization fields, and nullable product fields were tested without obtaining a successful response.
- The exception occurs during Entity Framework `SaveChanges()`. The response does not expose `EntityValidationErrors`, so the invalid entity/property is unknown.
- Omitting a numeric property may produce zero when its server-side type is non-nullable; omission does not prove that field is unrelated to the failure.
- Read requests succeeded with HTTP 200 and `Status.resCode: 0`. Pages 1, 2, and 3 each returned 20 records using the 2020-01-01 to 2026-11-09 date range. Page 2 returned `TenderItemId: 0` throughout, with funding source values 1, 2, or null. These observations informed test 5.
- `SendData` is documented with operation ID `TenderCompletion_CompleteTenderProfile`; its request model has no tender title or draft attachment field. `Organization.Title` is the organization name, and `ProductTitle` is a product name; neither is a tender title. The test markers in descriptions and product names do not establish a draft title.
- The documented API contains no explicit create-draft endpoint. The failed tests do not establish whether `SendData` can create drafts.
- Ask the backend developer for the detailed validation errors, a successful `SendData` request example, and the supported create-draft endpoint/schema including the tender title.
