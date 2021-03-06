# SOQL Parser JS

[![build](https://api.travis-ci.com/paustint/soql-parser-js.png?branch=master)](https://travis-ci.com/paustint/soql-parser-js)
[![npm version](https://badge.fury.io/js/soql-parser-js.svg)](https://badge.fury.io/js/soql-parser-js)
[![dependencies](https://david-dm.org/paustint/soql-parser-js.svg)](https://david-dm.org/paustint/soql-parser-js)

## Description
SOQL Parser JS will parse a SOQL query string into an object that is easy to work with and has the query broken down into usable parts.

This works in the browser as long as npm is used to install the package with dependencies and the browser supports ES6 or a transpiler is used.

*Warning*: antlr4 is a large library and is required for the parser to function, use in the browser with care.

## Examples
For an example of the parser, check out the [example application](https://paustint.github.io/soql-parser-js/).

## Usage

### Available functions
1. `parseQuery(soqlQueryString, options)`
2. `isQueryValid(SoqlQuery, options)`
3. `composeQuery(SoqlQuery, options)`

### Parse Query
The parser takes a SOQL query and returns structured data.

Options:
```typescript
export interface SoqlQueryConfig {
  continueIfErrors?: boolean; // default=false
  logging: boolean; // default=false
}
```

#### Typescript / ES6
```typescript
import { parseQuery } from 'soql-parser-js';

const soql = 'SELECT UserId, COUNT(Id) from LoginHistory WHERE LoginTime > 2010-09-20T22:16:30.000Z AND LoginTime < 2010-09-21T22:16:30.000Z GROUP BY UserId';

const soqlQuery = parseQuery(soql);

console.log(JSON.stringify(soqlQuery, null, 2));

```

#### Node
```javascript
var soqlParserJs = require('soql-parser-js');

const soql = 'SELECT UserId, COUNT(Id) from LoginHistory WHERE LoginTime > 2010-09-20T22:16:30.000Z AND LoginTime < 2010-09-21T22:16:30.000Z GROUP BY UserId';

const soqlQuery = soqlParserJs.parseQuery(soql);

console.log(JSON.stringify(soqlQuery, null, 2));
```

This yields an object with the following structure:

```json
{
  "fields": [
    {
      "text": "UserId"
    },
    {
      "fn": {
        "text": "COUNT(Id)",
        "name": "COUNT",
        "parameter": "Id"
      }
    }
  ],
  "subqueries": [],
  "sObject": "LoginHistory",
  "whereClause": {
    "left": {
      "field": "LoginTime",
      "operator": ">",
      "value": "2010-09-20T22:16:30.000Z"
    },
    "operator": "AND",
    "right": {
      "left": {
        "field": "LoginTime",
        "operator": "<",
        "value": "2010-09-21T22:16:30.000Z"
      }
    }
  },
  "groupBy": {
    "field": "UserId"
  }
}
```
### Check if Query is Valid
This will parse the AST tree to confirm the syntax is valid, but will not parse the tree into a data structure.
This method is faster than parsing the full query.

Options:
```typescript
export interface ConfigBase {
  logging: boolean; // default=false
}
```

```typescript
import { isQueryValid } from 'soql-parser-js';

const soql = 'SELECT UserId, COUNT(Id) from LoginHistory WHERE LoginTime > 2010-09-20T22:16:30.000Z AND LoginTime < 2010-09-21T22:16:30.000Z GROUP BY UserId';

const isValid = isQueryValid(soql);

console.log('isValid', isValid);

```

#### Node
```javascript
var soqlParserJs = require('soql-parser-js');

const soql = 'SELECT UserId, COUNT(Id) from LoginHistory WHERE LoginTime > 2010-09-20T22:16:30.000Z AND LoginTime < 2010-09-21T22:16:30.000Z GROUP BY UserId';

const isValid = isQueryValid(soql);

console.log('isValid', isValid);
```

### Compose Query
Composing a query turns a parsed query back into a SOQL query. For some operators, they may be converted to upper case (e.x. NOT, AND)

Options:
```typescript
export interface SoqlComposeConfig {
  logging: boolean; // default=false
  format: boolean; // default=false
  formatOptions?: {
    numIndent?: number; // default=1
    fieldMaxLineLen?: number; // default=60
    fieldSubqueryParensOnOwnLine?: boolean; // default=true
    whereClauseOperatorsIndented?: boolean; // default=false
  }
}
```

#### Typescript / ES6
```typescript
import { composeQuery } from 'soql-parser-js';

const soqlQuery = {
  fields: [
    {
      text: 'UserId',
    },
    {
      fn: {
        text: 'COUNT(Id)',
        name: 'COUNT',
        parameter: 'Id',
      },
    },
  ],
  subqueries: [],
  sObject: 'LoginHistory',
  whereClause: {
    left: {
      field: 'LoginTime',
      operator: '>',
      value: '2010-09-20T22:16:30.000Z',
    },
    operator: 'AND',
    right: {
      left: {
        field: 'LoginTime',
        operator: '<',
        value: '2010-09-21T22:16:30.000Z',
      },
    },
  },
  groupBy: {
    field: 'UserId',
  },
};

const query = composeQuery(soqlQuery);

console.log(query);

```

This yields an object with the following structure:

```sql
SELECT UserId, COUNT(Id) from LoginHistory WHERE LoginTime > 2010-09-20T22:16:30.000Z AND LoginTime < 2010-09-21T22:16:30.000Z GROUP BY UserId
```

### Format Query
This function is provided as a convenience and will parse a query and compose the query with formatting options provided.
```typescript
import { formatQuery } from 'soql-parser-js';

const query = `SELECT Id, Name, AccountNumber, AccountSource, AnnualRevenue, BillingAddress, BillingCity, BillingCountry, BillingGeocodeAccuracy, ShippingStreet, Sic, SicDesc, Site, SystemModstamp, TickerSymbol, Type, Website, (SELECT Id, Name, AccountId, Amount, CampaignId, CloseDate, CreatedById, Type FROM Opportunities), (SELECT Id, Name, AccountNumber, AccountSource, AnnualRevenue, BillingAddress, Website FROM ChildAccounts) FROM Account WHERE Name LIKE 'a%' OR Name LIKE 'b%' OR Name LIKE 'c%'`;

const formattedQuery1 = formatQuery(query);
const formattedQuery2 = formatQuery(query, { fieldMaxLineLen: 20, fieldSubqueryParensOnOwnLine: false, whereClauseOperatorsIndented: true });
const formattedQuery3 = formatQuery(query, { fieldSubqueryParensOnOwnLine: true, whereClauseOperatorsIndented: true });


```

```sql
-- formattedQuery1
SELECT Id, Name, AccountNumber, AccountSource, AnnualRevenue,
	BillingAddress, BillingCity, BillingCountry, BillingGeocodeAccuracy,
	ShippingStreet, Sic, SicDesc, Site, SystemModstamp, TickerSymbol, Type,
	Website,
	(
		SELECT Id, Name, AccountId, Amount, CampaignId, CloseDate,
			CreatedById, Type
		FROM Opportunities
	),
	(
		SELECT Id, Name, AccountNumber, AccountSource, AnnualRevenue,
			BillingAddress, Website
		FROM ChildAccounts
	)
FROM Account
WHERE Name LIKE 'a%'
OR Name LIKE 'b%'
OR Name LIKE 'c%'

-- formattedQuery2
SELECT Id, Name,
	AccountNumber, AccountSource,
	AnnualRevenue, BillingAddress,
	BillingCity, BillingCountry,
	BillingGeocodeAccuracy, ShippingStreet,
	Sic, SicDesc, Site,
	SystemModstamp, TickerSymbol, Type,
	Website,
	(SELECT Id, Name,
		AccountId, Amount, CampaignId,
		CloseDate, CreatedById, Type
	FROM Opportunities),
	(SELECT Id, Name,
		AccountNumber, AccountSource,
		AnnualRevenue, BillingAddress,
		Website
	FROM ChildAccounts)
FROM Account
WHERE Name LIKE 'a%'
	OR Name LIKE 'b%'
	OR Name LIKE 'c%'


-- formattedQuery3
SELECT Id, Name, AccountNumber, AccountSource, AnnualRevenue,
	BillingAddress, BillingCity, BillingCountry, BillingGeocodeAccuracy,
	ShippingStreet, Sic, SicDesc, Site, SystemModstamp, TickerSymbol, Type,
	Website,
	(
		SELECT Id, Name, AccountId, Amount, CampaignId, CloseDate,
			CreatedById, Type
		FROM Opportunities
	),
	(
		SELECT Id, Name, AccountNumber, AccountSource, AnnualRevenue,
			BillingAddress, Website
		FROM ChildAccounts
	)
FROM Account
WHERE Name LIKE 'a%'
	OR Name LIKE 'b%'
	OR Name LIKE 'c%'
```

### Options

```typescript
export interface SoqlQueryConfig {
  continueIfErrors?: boolean; // default=false
  logging: boolean; // default=false
  includeSubqueryAsField: boolean; // default=true
}

export interface SoqlComposeConfig {
  logging: boolean; // default=false
  format: boolean; // default=false
  formatOptions?: FormatOptions;
}

export interface FormatOptions {
  numIndent?: number; // default=1
  fieldMaxLineLen?: number; // default=60
  fieldSubqueryParensOnOwnLine?: boolean; // default=true
  whereClauseOperatorsIndented?: boolean; // default=false
  logging?: boolean; // default=false
}
```

### Data Models
```typescript
type LogicalOperator = 'AND' | 'OR';
type Operator = '=' | '<=' | '>=' | '>' | '<' | 'LIKE' | 'IN' | 'NOT IN' | 'INCLUDES' | 'EXCLUDES';
type TypeOfFieldConditionType = 'WHEN' | 'ELSE';
type GroupSelector = 'ABOVE' | 'AT' | 'BELOW' | 'ABOVE_OR_BELOW';
type LogicalPrefix = 'NOT';
type ForClause = 'VIEW' | 'UPDATE' | 'REFERENCE';
type UpdateClause = 'TRACKING' | 'VIEWSTAT';

interface Query {
  fields: Field[];
  subqueries: Query[];
  sObject?: string;
  sObjectAlias?: string;
  sObjectPrefix?: string[];
  sObjectRelationshipName?: string;
  where?: WhereClause;
  limit?: number;
  offset?: number;
  groupBy?: GroupByClause;
  having?: HavingClause;
  orderBy?: OrderByClause | OrderByClause[];
  withDataCategory?: WithDataCategoryClause;
  for?: ForClause;
  update?: UpdateClause;
}

interface SelectStatement {
  fields: Field[];
}

interface Field {
  text?: string;
  alias?: string;
  objectPrefix?: string;
  relationshipFields?: string[];
  fn?: FunctionExp;
  subqueryObjName?: string; // populated if subquery
  typeOf?: TypeOfField;
}

interface TypeOfField {
  field: string;
  conditions: TypeOfFieldCondition[];
}

interface TypeOfFieldCondition {
  type: TypeOfFieldConditionType;
  objectType?: string; // not present when ELSE
  fieldList: string[];
}

interface WhereClause {
  left: Condition;
  right?: WhereClause;
  operator?: LogicalOperator;
}

interface Condition {
  openParen?: number;
  closeParen?: number;
  logicalPrefix?: LogicalPrefix;
  field?: string;
  fn?: FunctionExp;
  operator: Operator;
  value?: string | string[];
  valueQuery?: Query;
}

interface OrderByClause {
  field?: string;
  fn?: FunctionExp;
  order?: 'ASC' | 'DESC';
  nulls?: 'FIRST' | 'LAST';
}

interface GroupByClause {
  field: string | string[];
  type?: 'CUBE' | 'ROLLUP';
}

interface HavingClause {
  left: HavingCondition;
  right?: HavingClause;
  operator?: LogicalOperator;
}

interface HavingCondition {
  openParen?: number;
  closeParen?: number;
  field?: string;
  fn?: FunctionExp;
  operator: string;
  value: string | number;
}

interface FunctionExp {
  text?: string; // Count(Id)
  name?: string; // Count
  alias?: string;
  parameter?: string | string[];
  isAggregateFn?: boolean;
  fn?: FunctionExp; // used for nested functions FORMAT(MIN(CloseDate))
}

interface WithDataCategoryClause {
  conditions: WithDataCategoryCondition[];
}

interface WithDataCategoryCondition {
  groupName: string;
  selector: GroupSelector;
  parameters: string[];
}

```

## CLI Usage
The CLI can be used to parse a query or compose a previously parsed query back to SOQL.

**Examples:**
```shell
$ npm install -g soql-parser-js
$ soql --help
$ soql --query "SELECT Id FROM Account"
$ soql -query "SELECT Id FROM Account"
$ soql -query "SELECT Id FROM Account" -output some-output-file.json
$ soql -query "SELECT Id FROM Account" -json
$ soql -query some-input-file.txt
$ soql -compose some-input-file.json
$ soql -compose some-input-file.json
$ soql -compose some-input-file.json -output some-output-file.json
```

**Arguments:**
```
    --query,   -q       A SOQL query surrounded in quotes or a file path to a text file containing a SOQL query.
    --compose, -c       An escaped and quoted parsed SOQL JSON string or a file path to a text file containing a parsed query JSON object.
    --output,  -o       Filepath.
    --json,    -j       Provide all output messages as JSON.
    --debug,   -d       Print additional debug log messages.
    --help,    -h       Show this help message.
```

## Contributing
All contributions are welcome on the project. Please read the [contribution guidelines](https://github.com/paustint/soql-parser-js/blob/master/CONTRIBUTING.md).

## Special Thanks
* This library is based on the ANTLR4 grammar file [produced by Mulesoft](https://github.com/mulesoft/salesforce-soql-parser/blob/antlr4/SOQL.g4).
* The following repository also was a help to get things started: https://github.com/petermetz/antlr-4-ts-test
