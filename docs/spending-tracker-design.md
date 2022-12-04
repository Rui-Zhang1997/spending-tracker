# Spending Tracker Design Doc
## Purpose
Implement a service that is designed to accept CSVs (MVP) containing financial statements and be able to do the following:

0. Accepts a .toml file to configure import/export behaviors.
1. Encrypt all financial information using AES-GCM.
2. Provide basic spending visualizations.
3. Service is designed to be able to be easily deployed to GCP/AWS.
4. Honestly, I'm just using this to get better at Rust. Just use YNAB.

Data should be easily queriable from a CLI and, if desired spin up a basic localhost web UI.

## Detailed Design
Design is split between five different components:

1. Database API (BE)
    * Handles CRUD operations between the Application Component and the underlying database
    * Manages data encryption/decryption
2. Application (BE)
    * Provides basic API that handles requests from various the CLI and Web UI Component
    * Performs data manipulation
3. Visualizer (BE)
    * Performs data visualization
    * Accepts visualization operations from Web UI Component
    * Does not perform any data queries
4. CLI (FE)
    * Handles command line operations and calls Application Component for associated data
    * Will have basic presentation functionality (i.e. tables only)
5. Web UI (FE)
    * Spins up an http://localhost instance with the ability to:
        * Present data in tables
        * Present visualizations by making calls to Visualizer Component

### Database API Basics
The Database Component should only perform basic CRUD operations and should not perform any operations of any kind unless those
operations are directly related to CRUD.

The DAPI will be as follows:

```
enum SpendingType {
    Credit,
    Gas,
    Groceries,
    Rent,
    Restaurant,
    Subscriptions,
    Services,
    Shopping,
    Utilities,
}

enum UpdateStrategy {
    Merge,
    Replace,
}

struct FinData {
    id: UUID,
    date: Date,
    type: Vec<SpendingType>,
    merchant: string,
    description: string,
    amount: f64
}

struct Query {
    ids: Vec<UUID>,
    dateRange: (Date, Date),
    types: Vec<SpendingType>,
    merchants: Vec<Regex>,
    amountRange: (f64, f64),
}

BatchInsert(data: Vec<FinData>) -> (u32, Err)

Insert(data: FinData) -> Err

Get(q: Query) -> Vec<FinData>

Update(id: UUID, update: FinData, strategy: UpdateStrategy) -> (Vec<UUID>, Err)

BatchUpdate(q: Query, update: FinData, strategy: UpdateStrategy) -> (Vec<UUID>, Err)

DeleteByQuery(q: Query, dryRun: bool) -> (Vec<UUID>, Err)

DeleteById(id: UUID) -> Err
```

### Application
Application Component handles communication with FE components and the Database. Also handles performing
calculations/manipulations on requested data.