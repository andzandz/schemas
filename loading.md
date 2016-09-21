# Loading of Adverts

- All adverts coming onto system for first time will be given advert.approval:pending, yet they will have the advert.status:online, thus all adverts are put online prior to them being checked and possibly removed.
- When adverts are approved, they will be set to approval:approved and the significantly_updated_date will be updated, as this is the date field the Property Spy uses to find suitable adverts.

- Adverts 

- Adverts in the system have the following statuses:
    - advert.status:online && approval:pending
        - part of advertisers current portfolio
        - advert has been selected to advertise
        - content and data has not yet been checked for quality by IFP
        - can be found via the Search Engine, but possibly unable to be found in targeted searches due to missing information
        - unavailable to Property Spy
    - advert.status:online && approval:approved
        - part of advertisers current portfolio
        - advert has been selected to advertise
        - content / data quality is approved (or improved) by IFP
        - can be found via the Search Engine
        - available to Property Spy
        
    - advert.status:online && approval:deferred
        - part of advertisers current portfolio
        - advert has been selected to advertise
        - content and data is not yet complete and awaiting input from advertiser to improve it, before approval from IFP
        - can be found via the Search Engine, but unlikely to be found in targeted searches due to missing information
        - unavailable to Property Spy
        
    - advert.status:offline
        - part of advertisers current portfolio
        - advert has not been selected to advertise
        - cannot be found via the Search Engine
        - unavailable to Property Spy
        - minimal version of advert can be accessed directly by URL
    - advert.status:archived
        - part of advertisers old portfolio
        - advert is not able to be selected to advertise, as it is an old advert
        - cannot be found via the Search Engine
        - unavailable to Property Spy
        - minimal version of advert can be accessed directly by URL
        
    - advert.status:deleted
        - advert is deleted from the Search Engine Database
        - advert is retained in the Backing Database
    - advert.status:deleted && approval:denied
        - advert has been denied by IFP
        - advert is deleted from the Search Engine Database
        - advert is retained in the Backing Database

The queue from which the Loader will pull the adverts will be loaded by 3 separate systems:

1. Old IFP
2. Advert Import
3. Checker

Each system will load the adverts with a set combination of action and advert.status fields as follows:

## 1. Adverts coming from Old IFP

The adverts will be loaded with one of four states:

### A. Adverts to Insert or Update as 'Online'

```json
{
  "action": "upsert",
  "advert": {
    "status": "online"
  }
}
```

### B. Adverts to Insert or Update as 'Offline'

```json
{
  "action": "upsert",
  "advert": {
    "status": "offline"
  }
}
```

### C. Adverts to Insert or Update as 'Archived'

```json
{
  "action": "upsert",
  "advert": {
    "status": "archived"
  }
}
```

### D. Adverts to Delete

```json
{
  "action": "delete"
}
```

## 2. Adverts coming from Advert Import

The adverts will be loaded with one of three states:

### A. Adverts to Insert or Update as 'Online'

```json
{
  "action": "upsert",
  "advert": {
    "status": "online",
    "approval": "pending"
  }
}
```

### B. Adverts to Insert or Update as 'Offline'

```json
{
  "action": "upsert",
  "advert": {
    "status": "offline",
    "approval": "pending"
  }
}
```

### C. Adverts to Delete

```json
{
  "action": "delete"
}
```

## 3. Adverts coming from Checker

The adverts will be loaded with one of two states:

### A. Adverts to Insert or Update as 'Approved'

```json
{
  "action": "upsert",
  "advert": {
    "status": "online",
    "approval": "approved"
  }
}
```

### B. Adverts to Delete that have been Denied

```json
{
  "action": "delete",
  "advert": {
    "status": "deleted",
    "approval": "denied"
  }
}
```

Notes: Adverts will never come through from the Checker with the approval of 'pending', as this state represents the adverts which have not yet been checked.