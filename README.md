# Custom Spacer Gadget for Jira Cloud (Forge)

A custom Forge app that creates a configurable spacer gadget for Jira Cloud dashboards. This gadget helps align other gadgets and section headers by providing transparent vertical spacing.

## Features

- **Configurable Height**: Set custom height in pixels (minimum 1px)
- **Transparent Background**: Blends seamlessly with dashboard
- **Persistent Configuration**: Remembers your height setting between sessions
- **Easy Installation**: Built with Atlassian Forge for simple deployment

## Prerequisites

- Node.js (v16 or higher)
- npm
- Atlassian Forge CLI
- Jira Cloud site with admin access

## Installation & Setup

### Step 1: Verify Forge CLI Installation

Open **PowerShell** and verify your installation:

```powershell
node --version
npm --version
forge --version
```

If Forge CLI is missing, install it:

```powershell
npm install -g @forge/cli
```

### Step 2: Create the Forge App

In PowerShell, run:

```powershell
forge create
```

Answer the prompts **exactly** as follows:

| Prompt                                   | Response                       |
| ---------------------------------------- | ------------------------------ |
| Select or create a Developer Space       | `Create a new Developer Space` |
| Enter a name for your Developer Space    | `spacer-gadget`                |
| Enter a name for your app                | `Spacer Gadget`                |
| Do you accept the terms?                 | `Yes`                          |
| Select an Atlassian app or platform tool | `Jira`                         |
| Select a category                        | `UI Kit`                       |
| Select a template                        | `jira-dashboard-gadget`        |

After creation, navigate into the project:

```powershell
cd Spacer-Gadget
```

### Step 3: Copy Your App ID (Important!)

1. Open `manifest.yml` in your preferred text editor (Notepad, VS Code, etc.)
2. Copy the full `id:` line under `app:` (it looks like: `ari:cloud:ecosystem::app/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)
3. Keep this ID handy for the next step

### Step 4: Replace Project Files

#### 4A. Update `manifest.yml`

Replace the entire content of `manifest.yml` with:

```yaml
app:
  id: ari:cloud:ecosystem::app/PASTE-YOUR-REAL-APP-ID-HERE
  runtime:
    name: nodejs24.x

modules:
  jira:dashboardGadget:
    - key: spacer-gadget
      title: Spacer Gadget
      description: Blank spacer for aligning gadgets and section headers
      thumbnail: https://developer.atlassian.com/platform/forge/images/icons/issue-panel-icon.svg
      resource: main
      render: native
      edit:
        resource: main
        render: native

resources:
  - key: main
    path: src/frontend/index.jsx

permissions:
  scopes:
    - "read:jira-work"
```

**Important**: Replace `PASTE-YOUR-REAL-APP-ID-HERE` with the App ID you copied in Step 3.

#### 4B. Update `src/frontend/index.jsx`

Replace the entire content of `src/frontend/index.jsx` with:

```jsx
import React from "react";
import ForgeReconciler, {
  Text,
  useProductContext,
  Textfield,
  Form,
  Button,
  FormSection,
  FormFooter,
  Label,
  RequiredAsterisk,
  useForm,
  Box,
} from "@forge/react";
import { view } from "@forge/bridge";

const FIELD_HEIGHT = "height";

// Inner form component — receives the resolved initial height as a prop.
// Pulling this into its own component (and keying it by initialHeight in
// the parent) guarantees useForm's defaultValues are correct on mount.
const EditForm = ({ initialHeight }) => {
  const { handleSubmit, register, getFieldId } = useForm({
    defaultValues: { [FIELD_HEIGHT]: initialHeight },
  });

  const onSubmit = (data) => {
    const height = parseInt(data[FIELD_HEIGHT], 10) || 120;
    const finalHeight = Math.max(1, height);
    view.submit({ [FIELD_HEIGHT]: finalHeight });
  };

  return (
    <Form onSubmit={handleSubmit(onSubmit)}>
      <FormSection>
        <Label labelFor={getFieldId(FIELD_HEIGHT)}>
          Height (pixels)
          <RequiredAsterisk />
        </Label>
        <Textfield
          {...register(FIELD_HEIGHT, { required: true, valueAsNumber: true })}
          type="number"
          placeholder="e.g. 150"
        />
        <Text>
          Minimum 1 pixel. Note: Jira's gadget chrome (title bar) is always
          shown around the spacer.
        </Text>
      </FormSection>
      <FormFooter>
        <Button appearance="primary" type="submit">
          Save configuration
        </Button>
      </FormFooter>
    </Form>
  );
};

export const Edit = () => {
  const context = useProductContext();

  // useProductContext can return a context object before
  // extension.gadgetConfiguration is populated, so wait for the extension
  // payload to be present. For brand-new gadgets, gadgetConfiguration will
  // be an empty object (still truthy) — that's the signal it's ready.
  const gadgetConfiguration = context?.extension?.gadgetConfiguration;
  if (!context || !context.extension || gadgetConfiguration === undefined) {
    return <Text>Loading...</Text>;
  }

  const rawHeight = gadgetConfiguration[FIELD_HEIGHT];
  const initialHeight = rawHeight != null ? Number(rawHeight) : 120;

  // key ensures EditForm remounts (and useForm re-seeds its defaults) if
  // the resolved initial height ever changes after first render.
  return <EditForm key={initialHeight} initialHeight={initialHeight} />;
};

const View = () => {
  const context = useProductContext();
  const config = context?.extension?.gadgetConfiguration || {};
  const height = parseInt(config[FIELD_HEIGHT], 10) || 120;

  return (
    <Box
      xcss={{
        height: `${height}px`,
        minHeight: `${height}px`,
        width: "100%",
        backgroundColor: "transparent",
      }}
    />
  );
};

const App = () => {
  const context = useProductContext();
  if (!context) return <Text>Loading...</Text>;

  return context.extension.entryPoint === "edit" ? <Edit /> : <View />;
};

ForgeReconciler.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
);
```

### Step 5: Deploy and Install

Run these commands in order:

```powershell
forge lint
forge deploy
forge install
```

During the `forge install` process, you'll be prompted to:

1. Select **Jira**
2. Enter your Jira site (e.g., `pryardley.atlassian.net`)
3. Confirm the required scopes (`read:jira-work`) → **Yes**

**Optional** (recommended after initial deploy):

```powershell
forge install --upgrade
```

### Step 6: Use the Spacer Gadget

1. Navigate to your Jira Dashboard
2. Click **Edit** (usually in the top-right corner)
3. Click **Add gadget**
4. Search for **"Spacer Gadget"**
5. Add it to your dashboard (typically under Rich Text section headers)
6. Click the **⋯** menu on the gadget → **Configure**
7. Set your desired height in pixels
8. Click **Save**

## Troubleshooting

### Configuration Not Persisting

If the height field always shows 120 instead of your saved value:

- Ensure you've completed Step 4 (copying the real App ID)
- Verify the App ID in `manifest.yml` matches exactly
- Run `forge deploy` and `forge install --upgrade` again

### Forge CLI Issues

If you encounter Forge CLI errors:

- Update to the latest version: `npm install -g @forge/cli@latest`
- Check Node.js version compatibility (requires v16+)

### Gadget Not Appearing

If the Spacer Gadget doesn't appear in the gadget list:

- Ensure you're logged into the correct Jira site
- Wait a few minutes after deployment for Atlassian to process the app
- Try clearing your browser cache

## Development

To run the app in development mode with hot-reloading:

```powershell
forge tunnel
```

This will open a tunnel to your local development environment. Press `Ctrl+C` to stop.

## License

This project is provided as-is for personal use.

## Support

For issues or questions:

1. Check the [Atlassian Forge documentation](https://developer.atlassian.com/platform/forge/)
2. Review the troubleshooting section above
3. Verify your Forge CLI is up to date
