# Snoopynet Test Authoring Standard

This document defines the standard for authoring new Snoopynet C# tests that use ArcObjects to validate product functionality.

Snoopynet tests are issue-based C# harness tests that run against supported workspace types, primarily Mobile GDB, with support for File GDB and GeoPackage where applicable.

## Naming Standard

Tests authored for this repository must use the following naming convention:

```text
Issue_XXXXX
```

Where `XXXXX` is the issue or bug tracking number.

The folder, namespace, class, and primary test method should follow the same issue naming pattern where practical.

Example:

```text
Issue_16405
Issue_16405_Test
```

## Required Bug Tracking Metadata

Every test must be associated with a bug tracking ID.

Each test class must include XML summary metadata containing:

- Issue number
- Short description
- Devtopia link

Example:

```csharp
/// <summary>
/// Issue 16405: Short description.
/// Devtopia: gh:ArcGISPro/.../issues/16405
/// </summary>
```

## Required Test Structure

Each test must include:

1. Static variables region
2. `Main` method
3. Identification and licensing region
4. Workspace acquisition
5. Workspace iteration
6. `InitTest`
7. Test method
8. `ShutdownTest`
9. Cleanup
10. Logger shutdown

## Static Variables

Each test must include a static variables region.

Required pattern:

```csharp
#region Static Variables
private static List<IWorkspace> workspaces = null;
private static List<string> datasetNames = null;
private static IWorkspace workspace = null;
private static IWorkspaceEdit workspaceEdit = null;
#endregion
```

Additional variables may be added as needed by the test.

## Identification and Licensing

Every test must initialize culture, identify the test, initialize licensing, and configure workspace skips as needed.

Required pattern:

```csharp
#region Identification and Licensing
try
{
  Resources.Culture = Thread.CurrentThread.CurrentCulture;

  QALogger.IdentifyTest(Resources.SourceFile, Resources.TestDescription);

  QAHelper.InitializeLicense();

  // Configure workspace skips as needed.
  QAHelper.SkipSde = true;
}
catch (Exception)
{
  try { QALogger.CloseLogFile(); }
  catch (Exception) { }
  return;
}
#endregion
```

Workspace skip behavior may also be controlled by `snoop.xml`.

## Supported Workspace Types

Tests should be written to support multiple workspace types unless the issue specifically requires otherwise.

Primary supported workspace types:

- Mobile GDB
- File GDB
- GeoPackage

Workspace skips must be intentional and documented in the test or configured through `snoop.xml`.

## Workspace Acquisition

Tests must acquire workspaces through `WorkspaceFunctions.GetDefaultWorkspaces`.

Required pattern:

```csharp
workspaces = WorkspaceFunctions.GetDefaultWorkspaces(null, Resources.TestDataPath, null);
```

## Workspace Iteration

Tests must iterate through all returned workspaces.

Required pattern:

```csharp
foreach (IWorkspace workspace in workspaces)
{
  InitTest(workspace);

  Issue_XXXXX_Test();

  ShutdownTest();
}
```

## InitTest

Every test must include an `InitTest` method.

Required pattern:

```csharp
private static void InitTest(IWorkspace workspace)
{
  #region Template-Defined Initialization
  Issue_XXXXX.workspace = workspace;
  workspaceEdit = (IWorkspaceEdit)workspace;
  WorkspaceFunctions.InitTest(workspace);
  #endregion

  #region Test-Specific Initialization
  #endregion
}
```

## ShutdownTest

Every test must include a `ShutdownTest` method.

Required pattern:

```csharp
private static void ShutdownTest()
{
  #region Template-Defined Shutdown
  WorkspaceFunctions.ShutdownTest(workspace);
  #endregion

  #region Test-Specific Shutdown
  GC.Collect();
  #endregion
}
```

## Edit Session Standard

Tests that modify data must use this edit session pattern:

```csharp
workspaceEdit.StartEditing(true);
workspaceEdit.StartEditOperation();

// Perform edit operation.

workspaceEdit.StopEditOperation();
workspaceEdit.StopEditing(true);
```

## Edit Session Exception Handling

If an exception occurs during an edit operation, abort the edit operation and stop editing without saving.

Recommended pattern:

```csharp
bool editOperationStarted = false;
bool editingStarted = false;

try
{
  workspaceEdit.StartEditing(true);
  editingStarted = true;

  workspaceEdit.StartEditOperation();
  editOperationStarted = true;

  // Perform edit operation.

  workspaceEdit.StopEditOperation();
  editOperationStarted = false;

  workspaceEdit.StopEditing(true);
  editingStarted = false;
}
catch
{
  if (editOperationStarted)
  {
    workspaceEdit.AbortEditOperation();
  }

  if (editingStarted)
  {
    workspaceEdit.StopEditing(false);
  }

  throw;
}
```

## Cleanup Standard

Every test must clean up workspaces and release resources.

Required cleanup includes:

- `WorkspaceFunctions.CleanupLocalWorkspaces(workspaces)`
- `WorkspaceFunctions.ShutdownTest(workspace)`
- `Marshal.ReleaseComObject(...)`
- `GC.Collect()`
- `QALogger.CloseLogFile()`

Required workspace cleanup pattern:

```csharp
try
{
  if (workspaces != null)
  {
    WorkspaceFunctions.CleanupLocalWorkspaces(workspaces);
  }
}
catch (Exception exc)
{
  QALogger.LogException(exc, "An error occurred while cleaning up the workspaces.");
}

QALogger.CloseLogFile();
```

## ArcObjects COM Release Standard

Any ArcObjects COM object opened or created within the test must be released.

Examples include:

- `ICursor`
- `IRow`
- `IFeatureClass`
- `IQueryDef2`
- `ITable`

Required release pattern:

```csharp
if (cursor != null)
{
  while (Marshal.ReleaseComObject(cursor) > 0) { }
}
```

A shared `ReleaseComObject` helper is preferred when repeated release logic is needed.

## QALogger.LogResult Standard

Each validation step must include at least one `QALogger.LogResult`.

Each result should include:

- Test step name
- `success` boolean
- Comment
- Issue link when applicable

Standard pattern:

```csharp
QALogger.LogResult("Issue_XXXXX_Test", success, comment);
```

Exception pattern:

```csharp
QALogger.LogResult("Issue_XXXXX_Test", exc, "gh:ArcGISPro/.../issues/XXXXX");
```

## Log Comment Standard

Each validation comment must clearly state expected and actual values.

Required wording:

```text
Expected: expectedValue, Got: actualValue
```

Example:

```csharp
comment = $"OBJECTID {objectIdValue} field {fieldName} Expected: {expectedValue}, Got: {actualValue}";
QALogger.LogResult("Issue_XXXXX_Test", success, comment);
```

## Logging Granularity

Logging granularity may vary by test case.

At minimum, each validation step must log one `QALogger.LogResult`.

Tests may log:

- One result per row
- One result per validation group
- Both row-level and summary-level results

Use row-level logging when validating specific ObjectIDs or field values.

## Null Handling Standard

Database nulls must be checked using both `null` and `DBNull.Value`.

Database nulls must be logged as `"null"`.

Required pattern:

```csharp
object valueObj = row.get_Value(fieldIndex);
string value = (valueObj == null || valueObj == DBNull.Value) ? "null" : Convert.ToString(valueObj);
```

Recommended helper:

```csharp
private static string FormatValueForLog(object value)
{
  return (value == null || value == DBNull.Value) ? "null" : Convert.ToString(value);
}
```

## Resources.resx Standard

Field names, feature class names, test data paths, and test descriptions must come from `Resources.resx`.

Avoid hard-coded literals when the value represents test metadata or test data configuration.

Every test's `Resources.resx` must include:

- `SourceFile`
- `TestDescription`
- `TestDataPath`
- `FeatureClassName`
- `FieldName`

Additional resource keys may be added as needed.

Examples:

- `InputFieldName`
- `OutputFieldName`
- `TableName`
- `RelationshipClassName`

## Query and Field Index Pattern

When reading rows, retrieve field indexes from the cursor fields.

Example:

```csharp
IFields fields = cursor.Fields;
int oidFieldIndex = fields.FindField("OBJECTID");
int fieldIndex = fields.FindField(Resources.FieldName);
```

Required field indexes should be validated when appropriate.

## Expected and Actual Field Validation Pattern

Recommended row validation pattern:

```csharp
while ((row = cursor.NextRow()) != null)
{
  object objectIdValue = row.get_Value(oidFieldIndex);
  object expectedObj = row.get_Value(expectedFieldIndex);
  object actualObj = row.get_Value(actualFieldIndex);

  string expectedValue = FormatValueForLog(expectedObj);
  string actualValue = FormatValueForLog(actualObj);

  success = expectedValue.Equals(actualValue);

  comment = $"OBJECTID {objectIdValue} Expected: {expectedValue}, Got: {actualValue}";
  QALogger.LogResult("Issue_XXXXX_Test", success, comment);
}
```

## Test Data Cleanup

Test data is expected to be cleaned up after each run.

Tests must still follow good edit session and cleanup practices even when test data is deleted after execution.

## Validation Before Submission

Before submitting a new test, complete all of the following:

1. Run the test locally against supported workspace types.
2. Review QA log output.
3. Confirm there are no COM leaks.
4. Confirm test data cleanup succeeds.
5. Confirm the entire solution builds.
6. Confirm every validation step logs a meaningful `QALogger.LogResult`.
7. Confirm all required metadata is present.
8. Confirm all configurable names and paths come from `Resources.resx`.

## Standard Test Skeleton

```csharp
using ESRI.ArcGIS.Geodatabase;
using ESRI.QA.Geodatabase;
using ESRI.QA.QaLoggerDotNet;
using System;
using System.Collections.Generic;
using System.Runtime.InteropServices;
using System.Threading;

namespace Issue_XXXXX
{
  /// <summary>
  /// Issue XXXXX: Short description.
  /// Devtopia: gh:ArcGISPro/.../issues/XXXXX
  /// </summary>
  public class Issue_XXXXX
  {
    #region Static Variables
    private static List<IWorkspace> workspaces = null;
    private static List<string> datasetNames = null;
    private static IWorkspace workspace = null;
    private static IWorkspaceEdit workspaceEdit = null;
    #endregion

    [STAThread]
    public static void Main(string[] args)
    {
      #region Identification and Licensing
      try
      {
        Resources.Culture = Thread.CurrentThread.CurrentCulture;

        QALogger.IdentifyTest(Resources.SourceFile, Resources.TestDescription);

        QAHelper.InitializeLicense();

        // Configure workspace skips as needed.
      }
      catch (Exception)
      {
        try { QALogger.CloseLogFile(); }
        catch (Exception) { }
        return;
      }
      #endregion

      try
      {
        workspaces = WorkspaceFunctions.GetDefaultWorkspaces(null, Resources.TestDataPath, null);

        foreach (IWorkspace workspace in workspaces)
        {
          InitTest(workspace);

          Issue_XXXXX_Test();

          ShutdownTest();
        }
      }
      catch (Exception exc)
      {
        QALogger.LogException(exc, "An unhandled error has occurred.");
      }

      try
      {
        if (workspaces != null)
        {
          WorkspaceFunctions.CleanupLocalWorkspaces(workspaces);
        }
      }
      catch (Exception exc)
      {
        QALogger.LogException(exc, "An error occurred while cleaning up the workspaces.");
      }

      QALogger.CloseLogFile();
    }

    private static void InitTest(IWorkspace workspace)
    {
      #region Template-Defined Initialization
      Issue_XXXXX.workspace = workspace;
      workspaceEdit = (IWorkspaceEdit)workspace;
      WorkspaceFunctions.InitTest(workspace);
      #endregion

      #region Test-Specific Initialization
      #endregion
    }

    private static void ShutdownTest()
    {
      #region Template-Defined Shutdown
      WorkspaceFunctions.ShutdownTest(workspace);
      #endregion

      #region Test-Specific Shutdown
      GC.Collect();
      #endregion
    }

    public static void Issue_XXXXX_Test()
    {
      try
      {
        bool success = false;
        string comment = string.Empty;

        // Test implementation.

        QALogger.LogResult("Issue_XXXXX_Test", success, comment);
      }
      catch (Exception exc)
      {
        QALogger.LogResult("Issue_XXXXX_Test", exc, "gh:ArcGISPro/.../issues/XXXXX");
      }
    }

    private static string FormatValueForLog(object value)
    {
      return (value == null || value == DBNull.Value) ? "null" : Convert.ToString(value);
    }
  }
}
```
