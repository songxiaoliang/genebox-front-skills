---
name: create-api
description: |
  自动生成 GeneBox React Native 项目的 API 接口文件，包括 Redux、Sagas 以及相关的注册配置。

  **When to use this skill:**
  - User types /create-api command followed by an API endpoint
  - User asks to "创建接口" or "添加新接口" for the GeneBox project
  - User mentions creating Redux and Sagas files for a new API endpoint
  - User needs to set up the complete API integration workflow

  This skill automates the complete workflow: parse URL → generate names → create Redux file → create Sagas file → register in index.js → configure RequestActions.js.

  **Command format:**
  - `/create-api /gene/homepage/gene/report/page` - Create single API endpoint (auto-generate responseKey)
  - `/create-api /api/user apiUser` - **Simplified syntax**: endpoint followed by responseKey (no `responseKey=` needed)
  - `/create-api /api/user apiUser category=Report` - Simplified + category
  - `/create-api /api/user apiUser User` - Even shorter: endpoint responseKey category
  - `/create-api /api/endpoint1 key1 /api/endpoint2 key2` - Batch create with simplified syntax

compatibility: |
  - GeneBox React Native project structure
  - Requires existing App/Redux and App/Sagas directories
  - Uses reduxsauce and seamless-immutable patterns
---

# Create API Skill

## Overview

This skill automates the creation of API integration files for GeneBox React Native projects:
1. Parse the API endpoint URL
2. Generate camelCase names for actions and files
3. Create Redux file with actions, types, and reducer
4. Create Sagas file with request handlers
5. Register reducer in `App/Redux/index.js`
6. Register sagas in `App/Sagas/index.js` (imports + takeLatest)
7. Export actions in `App/Redux/RequestActions.js`

## Workflow

### Batch Mode vs Single Mode

This skill supports both single API creation and batch creation:

**Single Mode:** One API endpoint at a time with full interactive prompts
**Batch Mode:** Multiple API endpoints at once with streamlined prompts

### Batch Mode Parsing

When user provides multiple API endpoints (e.g., `/create-api /api/one key1 /api/two key2`):

1. Parse all endpoints from the command (words starting with `/`)
2. For each endpoint, the next word is treated as responseKey (if it doesn't start with `/` or `category=`)
3. For each endpoint, extract optional parameters:
   - `responseKey=value` - Explicit response key format
   - `category=CategoryName` - Category for this specific endpoint
4. If category not specified for an endpoint, use the same category as the previous one, or default to "Report"

**Example batch commands:**
```
/create-api /gene/homepage/gene/report/page pageKey /gene/homepage/top/summary summaryKey /gene/homepage/gene/report/attention attentionKey
/create-api /api/one key1 category=Report /api/two key2 category=User
/create-api /api/user userKey User /api/order orderKey Order
```

### Batch Mode Workflow

When batch mode is detected (multiple endpoints):

### Step B1: Parse all endpoints

Extract from command:
- **API endpoints list**: All paths starting with `/`
- **Per-endpoint options**: `responseKey` and `category` that follow each endpoint

### Step B2: Select shared category (optional)

If all endpoints should use the same category, ask once:
"These endpoints will all use the same category. Select category (or choose different categories per endpoint):"
- Use same category for all
- Specify different categories

### Step B3: Process each endpoint

For each endpoint:
1. Generate names (camelCase, PascalCase, SCREAMING_SNAKE_CASE)
2. Check if files already exist
3. Collect all information before creating any files

### Step B4: Confirm batch creation

Show summary table:
```
Ready to create {N} APIs:

┌────────────────────────────────────┬──────────┬─────────────────────────┐
│ Endpoint                           │ Category │ Files will be created     │
├────────────────────────────────────┼──────────┼─────────────────────────┤
│ /gene/homepage/gene/report/page    │ Report   │ GeneHomepage...Redux.js │
│ /gene/homepage/top/summary         │ Report   │ GeneHomepage...Redux.js │
│ /gene/homepage/gene/report/att...  │ Report   │ GeneHomepage...Redux.js │
└────────────────────────────────────┴──────────┴─────────────────────────┘
```

Ask user to confirm before proceeding.

### Step B5: Create all files and registrations

Create all Redux and Sagas files first, then batch update the index files:

1. Create all Redux files
2. Create all Sagas files
3. Update `App/Redux/index.js` with all reducers
4. Update `App/Sagas/index.js` with all imports and takeLatest
5. Update `App/Redux/RequestActions.js` with all action exports

### Step B6: Display batch results

```
✅ Successfully created {N} API integrations!

📁 Files created:
[Table of all created files]

📝 All registrations updated:
- App/Redux/index.js (+{N} reducers)
- App/Sagas/index.js (+{N} ActionTypes, +{N*3} sagas, +{N*3} takeLatest)
- App/Redux/RequestActions.js (+{N} actions)

⚠️  Existing files skipped:
[If any files already existed, list them here]
```

### Single Mode Workflow

When only one endpoint is provided, follow the detailed interactive workflow below.

### Step 0: Parse input

Extract from user command:
- **API endpoint** (required): e.g., `/gene/homepage/gene/report/page`
- **responseKey** (optional, positional): The word immediately after the endpoint, e.g., `/api/user apiUser` → responseKey=`apiUser`
  - If not provided, auto-generate from URL
  - Also support explicit format: `responseKey=xxx`
- **category** (optional): `category=CategoryName` or as third positional argument

### Step 1: Ask user to select category

First, scan `App/Redux/` and `App/Sagas/` directories to get all existing category folders. Use the `ask_user_question` tool to let user select a category:

**Available categories:**
- AIChat
- Address
- CameraEffect
- CoinMall
- CouplePredict
- Discount
- GeneCard
- GeneMeow
- Insurance
- Lettery
- Mall
- Medicine
- Nutrition
- OneKey
- OnekeyUnlock
- Order
- Points
- Promotion
- Public
- Report
- Search
- Social
- User
- **+ Create new category**

Present these options to user with descriptions:
- **Report**: Report-related APIs (genes, phenotypes, analysis)
- **User**: User management APIs (login, profile, settings)
- **Social**: Social features APIs (feeds, comments, follows)
- **Public**: Public/common APIs (ads, banners, configs)
- **Mall**: Shopping/ordering APIs (products, orders, payments)
- **OneKey**: OneKey privilege APIs
- **GeneMeow**: Pet/gamification APIs
- **GeneCard**: Gene card APIs
- **AIChat**: AI chat APIs
- **CameraEffect**: Camera and video APIs
- **CouplePredict**: Couple prediction APIs
- **Points**: Points and rewards APIs
- **Promotion**: Promotion and marketing APIs
- **Address**: Address management APIs
- **Search**: Search APIs
- **Discount**: Coupon and discount APIs
- **Order**: Order management APIs
- **Medicine**: Medicine-related APIs
- **Insurance**: Insurance APIs
- **Lettery**: Lottery APIs
- **CoinMall**: Coin mall APIs
- **Nutrition**: Nutrition APIs
- **Activity**: Activity/marketing APIs
- **+ Create new**: Create a new category directory

If user selects "+ Create new", ask for the new category name and create the directory in both `App/Redux/` and `App/Sagas/`.

### Step 2: Ask for responseKey (optional)

Ask user: "Please specify the responseKey for extracting data from API response (e.g., 'homepageGeneReportPage'), or leave empty to auto-generate from URL."

If user provides a value, use it. Otherwise, auto-generate from the last segment of the URL path.

### Step 3: Generate names

Convert URL path to camelCase names:

**Pattern:**
- Remove leading `/`
- Split by `/`
- Convert each segment to camelCase
- Join together
- Result should start with lowercase

**Examples:**
- `/gene/homepage/gene/report/page` → `geneHomepageGeneReportPage`
- `/user/ws/public/match/record` → `userWsPublicMatchRecord`
- `/public/start/check` → `publicStartCheck`

**File naming convention:**
- Redux file: `{PascalCaseName}Redux.js`
- Sagas file: `{PascalCaseName}Sagas.js`

**Action names (camelCase):**
- `{name}` - Main action
- `{name}Success` - Success action
- `{name}Failure` - Failure action
- `{name}Clear` - Clear action

**ActionTypes (SCREAMING_SNAKE_CASE):**
- `{SCREAMING_SNAKE_NAME}`
- `{SCREAMING_SNAKE_NAME}_SUCCESS`
- `{SCREAMING_SNAKE_NAME}_FAILURE`
- `{SCREAMING_SNAKE_NAME}_CLEAR`

### Step 4: Create Redux file

Create file at: `App/Redux/{category}/{PascalCaseName}Redux.js`

**Template:**
```javascript
import { createReducer, createActions } from 'reduxsauce';
import Immutable from 'seamless-immutable';
/* ------------- Types and Action Creators ------------- */

// requestUrl: {apiEndpoint}

const { Types, Creators } = createActions({
    {camelCaseName}: ['requestParams', 'requestConfigs'],
    {camelCaseName}Success: ['responseData', 'requestParams'],
    {camelCaseName}Failure: ['status', 'msg'],
    {camelCaseName}Clear: null,
});

export const {PascalCaseName}ActionTypes = Types;
export default Creators;

/* ------------- Initial State ------------- */

export const INITIAL_STATE = Immutable({
    fetch: false,
    responseState: {
        status: -9999,
        msg: '',
    },
    data: {},
});

/* ------------- Selectors ------------- */

// add selectors if needed
export const {PascalCaseName}Selectors = {
    select{PascalCaseName}: (state) => state.{camelCaseName}?.data,
};

/* ------------- Reducers ------------- */

const {camelCaseName} = (state) => state.merge({ fetch: true });

const {camelCaseName}Success = (state, { responseData }) => state.merge({
    fetch: false,
    responseState: {
        status: responseData.status,
        msg: responseData.msg,
    },
    data: responseData.data?.replace?.{responseKey} || {},
});

const {camelCaseName}Failure = (state, { status, msg }) => state.merge({
    fetch: false,
    responseState: {
        status: status,
        msg: msg,
    },
    data: {},
});

const {camelCaseName}Clear = (state) => state.merge({
    data: {},
});


/* ------------- Hookup Reducers To Types ------------- */

export const reducer = createReducer(INITIAL_STATE, {
    [Types.{SCREAMING_SNAKE_CASE_NAME}]: {camelCaseName},
    [Types.{SCREAMING_SNAKE_CASE_NAME}_SUCCESS]: {camelCaseName}Success,
    [Types.{SCREAMING_SNAKE_CASE_NAME}_FAILURE]: {camelCaseName}Failure,
    [Types.{SCREAMING_SNAKE_CASE_NAME}_CLEAR]: {camelCaseName}Clear,
});
```

### Step 5: Create Sagas file

Create file at: `App/Sagas/{category}/{PascalCaseName}Sagas.js`

**Template:**
```javascript
import { put } from 'redux-saga/effects';
import {PascalCaseName}Actions from '../../Redux/{category}/{PascalCaseName}Redux';
import RequestActions from '../../Redux/RequestRedux';

function getDefaultRequestParams() {
    return {};
}

function getDefaultRequestConfigs() {
    return {
        requestUrl: '{apiEndpoint}',
        requestType: 'get',
        actions: {
            onResponseSuccess: {PascalCaseName}Actions.{camelCaseName}Success,
            onResponseFailure: {PascalCaseName}Actions.{camelCaseName}Failure,
        },
        isBlock: false,
        retryTimes: 1,
    };
}

export function* request{PascalCaseName}(action) {
    let requestParams = getDefaultRequestParams();
    if (action.requestParams) {
        requestParams = Object.assign(requestParams, action.requestParams);
    }
    let requestConfigs = getDefaultRequestConfigs();
    if (action.requestConfigs) {
        requestConfigs = Object.assign(requestConfigs, action.requestConfigs);
    }
    yield put(RequestActions.request(requestParams, requestConfigs));
}

export function* {camelCaseName}Success(action) {
    // do something when response succese
}

export function* {camelCaseName}Failure(action) {
    // do something when response Failed
}
```

### Step 6: Register in Redux index.js

Add to `App/Redux/index.js` in the reducers object:

```javascript
{camelCaseName}: require('./{category}/{PascalCaseName}Redux').reducer,
```

**Insert location:** Find a good alphabetical/logical position among existing reducers in the same category.

### Step 7: Register in Sagas index.js

Three edits needed in `App/Sagas/index.js`:

**A. Import ActionTypes** (with other ActionTypes imports):
```javascript
import { {PascalCaseName}ActionTypes } from '../Redux/{category}/{PascalCaseName}Redux';
```

**B. Import saga functions** (with other saga imports):
```javascript
import { request{PascalCaseName}, {camelCaseName}Success, {camelCaseName}Failure } from './{category}/{PascalCaseName}Sagas';
```

**C. Add takeLatest registrations** (in the all() array):
```javascript
takeLatest({PascalCaseName}ActionTypes.{SCREAMING_SNAKE_CASE_NAME}_FAILURE, {camelCaseName}Failure),
takeLatest({PascalCaseName}ActionTypes.{SCREAMING_SNAKE_CASE_NAME}_SUCCESS, {camelCaseName}Success),
takeLatest({PascalCaseName}ActionTypes.{SCREAMING_SNAKE_CASE_NAME}, request{PascalCaseName}),
```

**Order:** FAILURE, SUCCESS, then the main action type.

### Step 8: Configure RequestActions.js

Two edits needed in `App/Redux/RequestActions.js`:

**A. Import actions** (with other imports):
```javascript
import {PascalCaseName}Actions from './{category}/{PascalCaseName}Redux';
```

**B. Export in default object** (at the end of the object):
```javascript
{camelCaseName}: {PascalCaseName}Actions.{camelCaseName},
```

### Step 9: Display results

Show user a summary:
```
✅ API integration created successfully!

📁 Files created:
- App/Redux/{category}/{PascalCaseName}Redux.js
- App/Sagas/{category}/{PascalCaseName}Sagas.js

📝 Registrations added:
- App/Redux/index.js
- App/Sagas/index.js (ActionTypes, sagas, takeLatest)
- App/Redux/RequestActions.js

🎯 Usage:
dispatch(RequestActions.{camelCaseName}());
```

## Helper Functions

### Convert URL to camelCase name

```javascript
function urlToCamelCase(url) {
    // Remove leading slash and split by /
    const segments = url.replace(/^\//, '').split('/');
    
    // Convert each segment: lowercase first, then capitalize subsequent words
    return segments.map((segment, index) => {
        // Handle hyphenated or underscored words
        const words = segment.split(/[-_]/);
        return words.map((word, wordIndex) => {
            if (index === 0 && wordIndex === 0) {
                return word.toLowerCase();
            }
            return word.charAt(0).toUpperCase() + word.slice(1).toLowerCase();
        }).join('');
    }).join('');
}

// Examples:
// /gene/homepage/gene/report/page → geneHomepageGeneReportPage
// /user/ws/public/match/record → userWsPublicMatchRecord
```

### Convert camelCase to PascalCase

```javascript
function camelToPascal(camel) {
    return camel.charAt(0).toUpperCase() + camel.slice(1);
}

// geneHomepageGeneReportPage → GeneHomepageGeneReportPage
```

### Convert camelCase to SCREAMING_SNAKE_CASE

```javascript
function camelToScreamingSnake(camel) {
    return camel.replace(/[A-Z]/g, letter => `_${letter.toLowerCase()}`).toUpperCase();
}

// geneHomepageGeneReportPage → GENE_HOMEPAGE_GENE_REPORT_PAGE
```

## Edge Cases

**URL with version numbers:**
- `/api/v1/users` → `apiV1Users`
- `/api/v2/report/list` → `apiV2ReportList`

**URL with hyphens:**
- `/user/public-match/record` → `userPublicMatchRecord`

**Duplicate names:**
- Check if files already exist before creating
- If exists, ask user whether to overwrite or skip

**Category directories:**
- Scan existing directories in `App/Redux/` and `App/Sagas/`
- If user selects "+ Create new", create the directory in both locations
- If selected category doesn't exist, create it

## Example Usage

**User:** `/create-api /gene/homepage/gene/report/page responseKey=homepageGeneReportPage category=Report`

**Claude:**
> Creating API integration for `/gene/homepage/gene/report/page`...
>
> Generated names:
> - camelCase: `geneHomepageGeneReportPage`
> - PascalCase: `GeneHomepageGeneReportPage`
> - ActionTypes: `GENE_HOMEPAGE_GENE_REPORT_PAGE`
>
> Creating files and registering...
>
> ✅ Complete!
>
> 📁 Files:
> - App/Redux/Report/GeneHomepageGeneReportPageRedux.js
> - App/Sagas/Report/GeneHomepageGeneReportPageSagas.js
>
> 📝 Registered in:
> - App/Redux/index.js
> - App/Sagas/index.js
> - App/Redux/RequestActions.js
>
> 🎯 Usage: `dispatch(RequestActions.geneHomepageGeneReportPage());`
