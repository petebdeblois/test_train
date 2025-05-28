# 🖥️ Salesforce Custom Component Training: Complete Workflow

---

## 1. **Developer Environment Setup (Windows)**

### 1.1. Install Windows Terminal

1. Open the **Microsoft Store**.
2. Search for **Windows Terminal**.
3. Click **Get** or **Install**.
4. Once installed, launch it from the Start menu.

### 1.2. Install Git (with Git Bash)

1. Go to [git-scm.com/downloads](https://git-scm.com/downloads).
2. Download and run the Windows installer.

   - On “Choosing the default terminal emulator”, select **“Use Git Bash only”**.
   - Leave most settings as default.

3. Finish the installation.
4. Verify:

   ```bash
   git --version
   ```

   You can also open **Git Bash** from the Start menu.

### 1.3. Install Node.js

1. Go to [nodejs.org](https://nodejs.org/).
2. Download the **LTS** version.
3. Run the installer, follow prompts.
4. Verify:

   ```bash
   node -v
   npm -v
   ```

### 1.4. Install Visual Studio Code

- Download and install from [code.visualstudio.com](https://code.visualstudio.com/download).

### 1.5. Install Salesforce CLI

- Download and install from [developer.salesforce.com/tools/salesforcecli](https://developer.salesforce.com/tools/salesforcecli).

---

## 2. **Create and Set Up a Salesforce Project**

### 2.1. Open Windows Terminal

- Launch Windows Terminal from your Start menu.

### 2.2. Create a New Salesforce Project

```bash
sf project generate --name MySalesforceProject
```

_Replace `MySalesforceProject` with your project name._

### 2.3. Navigate into Your Project Folder

```bash
cd MySalesforceProject
```

### 2.4. Open Your Project in VS Code

- Go to **File > Open Folder...** and select your project folder.

### 2.5. Install VS Code Extensions

1. In VS Code, open the Extensions panel (square icon on the sidebar).
2. Install recommended Salesforce and related extensions.  
   Or add this to `.vscode/extensions.json`:

   ```json
   {
     "recommendations": [
       "salesforce.salesforcedx-vscode",
       "redhat.vscode-xml",
       "salesforce.salesforcedx-vscode-apex",
       "salesforce.salesforcedx-vscode-core",
       "salesforce.salesforcedx-vscode-visualforce",
       "salesforce.salesforcedx-vscode-lightning",
       "esbenp.prettier-vscode",
       "streetsidesoftware.code-spell-checker",
       "github.copilot"
     ]
   }
   ```

---

## 3. **Connect to Your Salesforce Developer Org**

### 3.1. Open the Command Palette

- Press `Ctrl + Shift + P` (Windows) or `Cmd + Shift + P` (Mac).

### 3.2. Authorize Org

- Type and select:
  ```
  Salesforce: Authorize Org
  ```
  _You might see `SFDX: Authorize an Org` — both work with the Salesforce Extension Pack._

### 3.3. Choose Org Type

- Select **Project Default** (or provide an alias).
- For login URL, select **Production** for Developer Orgs (use Sandbox if needed).

### 3.4. Log In

- Browser window opens; log in with Salesforce Developer Org credentials.
- On success, VS Code confirms authorization.

### 3.5. Verify Connection

```bash
sf org list
```

_You should see your org listed as default._

---

## 4. **Salesforce Project Folder Structure**

```
MySalesforceProject/
│
├── force-app/
│   └── main/
│       └── default/
│           ├── aura/
│           ├── lwc/
│           ├── classes/
│           ├── objects/
│           └── ... (other metadata types)
│
├── config/
│   └── project-scratch-def.json
│
├── scripts/
│   └── soql/
│
├── .sfdx/             # Local SFDX state (auto-generated, hidden)
├── .gitignore         # Git ignore file
├── package.json       # Project dependencies/config
├── sfdx-project.json  # Project config (name, paths, etc.)
└── README.md          # Documentation
```

_If you don’t see `aura/`, it will be created when you add your first Aura component._

---

## 5. **Create an Aura Component in VS Code**

### 5.1. Open the Command Palette

- Press `Ctrl + Shift + P` (Windows) or `Cmd + Shift + P` (Mac).

### 5.2. Run the Create Command

- Type and select:
  ```
  Salesforce: Create Aura Component
  ```
  _Or `SFDX: Create Aura Component`._

### 5.3. Provide Details

- Enter your component’s name (e.g., `CoveoCustom1`).
- Press Enter.

### 5.4. Choose Output Directory

- Browse to `force-app/main/default/aura/`.
  - If it doesn’t exist, create it or let VS Code do so.
- Click **Create Component Here**.

### 5.5. Review Files

VS Code generates:

- `CoveoCustom1.cmp`
- `CoveoCustom1Controller.js`
- `CoveoCustom1Helper.js`
- `CoveoCustom1Renderer.js`
- `CoveoCustom1.cmp-meta.xml`
- `CoveoCustom1.design`

---

## 6. **Add and Retrieve a Static Resource**

### 6.1. Create a Static Resource File

- Example: `customContext.js`

```js
window.coveoCustomScripts["default"] = function (promise, component) {
  $(".CoveoSearchInterface").on("changeAnalyticsCustomData", (e, args) => {
    console.log("It works, I'm alive");
    if (args.type === "ClickEvent") {
      args.metaObject.newCustomData = "newCustomDataValue"; //https://connect.coveo.com/s/article/15317
    }
  });
};
```

### 6.2. Upload as Static Resource

1. In Salesforce Setup, search for **Static Resources**.
2. Click **New**.
3. Enter a name (`customContext`), add a description if desired.
4. Browse and select your `.js` file, then save.

### 6.3. Retrieve Static Resource with SF CLI

```bash
sf project retrieve start --metadata StaticResource:customContext
```

---

## 7. **Example: Use Static Resource in Aura Component**

**Component (`CoveoCustom1.cmp`):**

```xml
<aura:component implements='forceCommunity:availableForAllPageTypes'>
  <aura:attribute name="name" type="String" default="" access="global" />
  <aura:attribute name="debug" type="Boolean" default="false" access="global" />
  <aura:attribute name="searchHub" type="String" default="" access="global" />
  <CoveoV2:CommunitySearch  name="{!v.name}"
                            debug="{!v.debug}"
                            customScripts="{!$Resource.customContext}"
                            searchHub="{!v.searchHub}"/>
</aura:component>
```

**Design File (`CoveoCustom1.design`):**

```xml
<design:component label="coveo1">
    <design:attribute name="name" label="Name" />
    <design:attribute name="debug" label="Debug" />
    <design:attribute name="searchHub" label="Search Hub" />
</design:component>
```

---

## 8. **Deploy to Salesforce**

To deploy your component and static resource:

```bash
sf project deploy start --source-dir force-app
```

---

## 9. **Test and Use Your Component**

1. Go to your Salesforce Community site.
2. Replace your search component in the builder with the new Aura component.
3. Publish changes.
4. Test by clicking a result—check the DevConsole for custom metadata from your script.

---
