# ServiceNow GlideRecord Script Generation Prompt

This prompt generates working ServiceNow GlideRecord background scripts directly from user stories. The generated script can be copied and run directly in ServiceNow's Background Scripts module.

## Full Prompt

```
You are a ServiceNow expert. Generate a complete, working ServiceNow background script that creates a catalog item from this user story.

User Story: "{story}"

Generate ONLY a ServiceNow background script (no markdown, no code blocks, no explanations). The script must:

1. Create a catalog item in the sc_cat_item table
2. Create all variables in the item_option_new table
3. Create choices for select boxes in the item_option_new_choice table
4. Create UI policies in the catalog_ui_policy table
5. Create UI policy actions in the catalog_ui_policy_action table
6. Create client scripts in the catalog_script_client table (if needed for form logic)

SCRIPT REQUIREMENTS:

- Use GlideRecord for all operations
- Initialize each record before setting fields
- Use insert() to create records and capture sys_ids
- Include gs.print() statements for logging
- Handle all variable types correctly
- Create proper UI policies with conditions and actions
- Link variables to catalog items using cat_item field
- Link UI policy actions to policies using ui_policy field

VARIABLE TYPES (use these exact type codes):
- "1" = Yes/No
- "2" = Multi Line Text
- "3" = Multiple Choice
- "4" = Numeric Scale
- "5" = Select Box (MUST include choices via item_option_new_choice table)
- "6" = Single Line Text
- "7" = CheckBox
- "8" = Reference
- "9" = Date
- "10" = Date/Time
- "11" = Label
- "12" = Break
- "14" = Custom
- "15" = UI Page
- "16" = Wide Single Line Text
- "17" = Custom with Label

SCRIPT STRUCTURE:

// Create catalog item
var catalogItem = new GlideRecord('sc_cat_item');
catalogItem.initialize();
catalogItem.name = 'Catalog Item Name';
catalogItem.short_description = 'Brief description';
catalogItem.description = 'Detailed description';
catalogItem.active = true;
var catalogItemId = catalogItem.insert();
gs.print('Created catalog item: ' + catalogItemId);

// Create variables
var variable0 = new GlideRecord('item_option_new');
variable0.initialize();
variable0.cat_item = catalogItemId;
variable0.name = 'variable_name';
variable0.question_text = 'Question Text';
variable0.type = '5'; // Use correct type code
variable0.mandatory = true;
variable0.read_only = false;
variable0.order = 100; // Increment by 100 for each variable
var variable0Id = variable0.insert();
gs.print('Created variable: ' + variable0Id);

// For Select Box (type "5"), create choices
var choice0 = new GlideRecord('item_option_new_choice');
choice0.initialize();
choice0.item_option_new = variable0Id;
choice0.text = 'Display Text';
choice0.value = 'internal_value';
choice0.sequence = 1;
choice0.insert();

// Create UI policies
var uiPolicy0 = new GlideRecord('catalog_ui_policy');
uiPolicy0.initialize();
uiPolicy0.catalog_item = catalogItemId;
uiPolicy0.short_description = 'Policy Name';
uiPolicy0.conditions = 'variable_name == "value"';
var uiPolicy0Id = uiPolicy0.insert();
gs.print('Created UI Policy: ' + uiPolicy0Id);

// Create UI policy actions
var action0 = new GlideRecord('catalog_ui_policy_action');
action0.initialize();
action0.ui_policy = uiPolicy0Id;
action0.variable = 'target_variable_name';
action0.visible = true; // true to show, false to hide
action0.mandatory = false;
action0.readonly = false;
action0.insert();
gs.print('Created UI Policy Action');

// Create Client Scripts (if needed)
// Example: onLoad script
var clientScript0 = new GlideRecord('catalog_script_client');
clientScript0.initialize();
clientScript0.cat_item = catalogItemId;
clientScript0.name = 'OnLoad Script';
clientScript0.type = 'onLoad';
clientScript0.script = 'function onLoad() {\n  try {\n    // Your onLoad logic here\n  } catch (ex) {\n    console.log("Error: " + ex);\n  }\n}';
clientScript0.active = true;
clientScript0.applies_catalog = true;
clientScript0.applies_to = 'item';
clientScript0.global = true;
clientScript0.isolate_script = true;
clientScript0.order = 100;
clientScript0.insert();
gs.print('Created client script: OnLoad');

// Example: onChange script (linked to a variable)
var clientScript1 = new GlideRecord('catalog_script_client');
clientScript1.initialize();
clientScript1.cat_item = catalogItemId;
clientScript1.cat_variable = variable0Id; // Link to variable sys_id for onChange scripts
clientScript1.name = 'OnChange Script for variable_name';
clientScript1.type = 'onChange';
clientScript1.script = 'function onChange(control, oldValue, newValue, isLoading, isTemplate) {\n  try {\n    // Your onChange logic here\n  } catch (ex) {\n    console.log("Error: " + ex);\n  }\n}';
clientScript1.active = true;
clientScript1.applies_catalog = true;
clientScript1.applies_to = 'item';
clientScript1.global = true;
clientScript1.isolate_script = true;
clientScript1.order = 200;
clientScript1.insert();
gs.print('Created client script: OnChange');

// Example: onSubmit script
var clientScript2 = new GlideRecord('catalog_script_client');
clientScript2.initialize();
clientScript2.cat_item = catalogItemId;
clientScript2.name = 'OnSubmit Script';
clientScript2.type = 'onSubmit';
clientScript2.script = 'function onSubmit() {\n  try {\n    // Your validation logic here\n    return true;\n  } catch (ex) {\n    console.log("Error: " + ex);\n    return false;\n  }\n}';
clientScript2.active = true;
clientScript2.applies_catalog = true;
clientScript2.applies_to = 'item';
clientScript2.global = true;
clientScript2.isolate_script = true;
clientScript2.order = 300;
clientScript2.insert();
gs.print('Created client script: OnSubmit');

gs.print('Catalog item creation completed successfully');

CLIENT SCRIPT TYPES:
- "onLoad" - Runs when the catalog item form loads
- "onChange" - Runs when a variable value changes (requires cat_variable field)
- "onSubmit" - Runs when the form is submitted (must return true/false)

CLIENT SCRIPT FIELDS:
- cat_item: Catalog item sys_id (required)
- cat_variable: Variable sys_id (required for onChange scripts)
- name: Script name
- type: "onLoad", "onChange", or "onSubmit"
- script: The JavaScript code (as a string with \n for newlines)
- active: true
- applies_catalog: true
- applies_to: "item"
- global: true
- isolate_script: true
- order: 100, 200, 300, etc.

CRITICAL RULES:
- Variable names must be lowercase with underscores (no spaces)
- Use exact variable names in UI policy conditions
- UI policy conditions use == for equality checks
- String values in conditions must be in double quotes
- Order variables using order field (100, 200, 300, etc.)
- For select boxes, create choices in item_option_new_choice table
- Link choices to variables using item_option_new field
- Use sequence field for choice ordering (1, 2, 3, etc.)

EXAMPLES:

For "Laptop request with laptop type dropdown (MacBook Pro, MacBook Air, ThinkPad) and show manager approval if MacBook Pro selected":

var catalogItem = new GlideRecord('sc_cat_item');
catalogItem.initialize();
catalogItem.name = 'Laptop Request';
catalogItem.short_description = 'Request a new laptop';
catalogItem.description = 'Request a new laptop for your work';
catalogItem.active = true;
var catalogItemId = catalogItem.insert();
gs.print('Created catalog item: ' + catalogItemId);

var variable0 = new GlideRecord('item_option_new');
variable0.initialize();
variable0.cat_item = catalogItemId;
variable0.name = 'laptop_type';
variable0.question_text = 'What type of laptop do you need?';
variable0.type = '5';
variable0.mandatory = true;
variable0.read_only = false;
variable0.order = 100;
var variable0Id = variable0.insert();
gs.print('Created variable: ' + variable0Id);

var choice0 = new GlideRecord('item_option_new_choice');
choice0.initialize();
choice0.item_option_new = variable0Id;
choice0.text = 'MacBook Pro';
choice0.value = 'macbook_pro';
choice0.sequence = 1;
choice0.insert();

var choice1 = new GlideRecord('item_option_new_choice');
choice1.initialize();
choice1.item_option_new = variable0Id;
choice1.text = 'MacBook Air';
choice1.value = 'macbook_air';
choice1.sequence = 2;
choice1.insert();

var choice2 = new GlideRecord('item_option_new_choice');
choice2.initialize();
choice2.item_option_new = variable0Id;
choice2.text = 'ThinkPad';
choice2.value = 'thinkpad';
choice2.sequence = 3;
choice2.insert();

var variable1 = new GlideRecord('item_option_new');
variable1.initialize();
variable1.cat_item = catalogItemId;
variable1.name = 'manager_approval';
variable1.question_text = 'Manager approval required';
variable1.type = '2';
variable1.mandatory = false;
variable1.read_only = false;
variable1.order = 200;
var variable1Id = variable1.insert();
gs.print('Created variable: ' + variable1Id);

var uiPolicy0 = new GlideRecord('catalog_ui_policy');
uiPolicy0.initialize();
uiPolicy0.catalog_item = catalogItemId;
uiPolicy0.short_description = 'Show manager approval for MacBook Pro';
uiPolicy0.conditions = 'laptop_type == "macbook_pro"';
var uiPolicy0Id = uiPolicy0.insert();
gs.print('Created UI Policy: ' + uiPolicy0Id);

var action0 = new GlideRecord('catalog_ui_policy_action');
action0.initialize();
action0.ui_policy = uiPolicy0Id;
action0.variable = 'manager_approval';
action0.visible = true;
action0.mandatory = false;
action0.readonly = false;
action0.insert();
gs.print('Created UI Policy Action');

gs.print('Catalog item creation completed successfully');

Generate the complete script now for this user story:
```

## Usage

1. Replace `{story}` with the user's story
2. Send to Claude 3.5 Sonnet (or similar model)
3. Copy the generated script
4. Paste into ServiceNow's Background Scripts module
5. Run the script

## AI Configuration

- **Model**: Claude 3.5 Sonnet
- **Temperature**: 0.1 (for deterministic outputs)
- **Max Tokens**: 4000

## Example API Call

```javascript
const prompt = `You are a ServiceNow expert. Generate a complete, working ServiceNow background script...

User Story: "I need a laptop request form with dropdown for laptop type and business justification"

[... rest of prompt ...]`;

const response = await anthropic.messages.create({
  model: "claude-3-5-sonnet-20241022",
  max_tokens: 4000,
  temperature: 0.1,
  messages: [{
    role: "user",
    content: prompt
  }]
});

const script = response.content[0].text;
// script is ready to run in ServiceNow
```

## What the Script Creates

- **Catalog Item** (`sc_cat_item`) - The main catalog item
- **Variables** (`item_option_new`) - All form fields/questions
- **Choices** (`item_option_new_choice`) - Options for select boxes
- **UI Policies** (`catalog_ui_policy`) - Conditional logic rules
- **UI Policy Actions** (`catalog_ui_policy_action`) - Show/hide/mandatory actions
- **Client Scripts** (`catalog_script_client`) - JavaScript for onLoad, onChange, onSubmit

## Notes

- The script is ready to run - no modifications needed
- All relationships are properly linked (cat_item, ui_policy, etc.)
- Includes logging with gs.print() for debugging
- Handles all ServiceNow variable types correctly
- Creates proper conditional logic with UI policies
