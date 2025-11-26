# MibTest Repository - Teil 2

## XML-Konfiguration

### ConfigMib.xml - Haupt-Konfiguration

Die `XmlDefinitions/ConfigMib.xml` ist die zentrale Konfigurationsdatei:

```xml
<Config>
  <!-- Security & Timing -->
  <SecurityNr>20103</SecurityNr>
  <ManualResetKeys>Menu + Wählrad + NO</ManualResetKeys>
  
  <!-- Logging -->
  <LogOptionsDiagTypes>Detailed</LogOptionsDiagTypes>
  <LogOptionsRequestAndResponse>Compact</LogOptionsRequestAndResponse>
  
  <!-- CANoe Configuration -->
  <CanOeConfig>CANoe\MIB3_Config.cfg</CanOeConfig>
  
  <!-- XML Schemas -->
  <DidSchema>XML\DidSchema.xsd</DidSchema>
  <DtcSchema>XML\DtcSchema.xsd</DtcSchema>
  <RoutineSchema>XML\RoutineSchema.xsd</RoutineSchema>
  <IoControlSchema>XML\IoControlSchema.xsd</IoControlSchema>
  <DatasetSchema>XML\DatasetSchema.xsd</DatasetSchema>
  
  <!-- DID Definitions -->
  <DidList>DidList_MIB_Identification.xml</DidList>
  <DidList>DidList_MIB_Messwert.xml</DidList>
  <DidList>DidList_MIB_Codierung.xml</DidList>
  <DidList>DidList_MIB_Anpassungen.xml</DidList>
  
  <!-- DTC Definitions -->
  <DtcList>DtcList_MIB.xml</DtcList>
  
  <!-- Routine Definitions -->
  <RoutineList>RoutineList_MIB.xml</RoutineList>
  
  <!-- IoControl Definitions -->
  <IoControlList>IoControlList_MIB.xml</IoControlList>
  
  <!-- Dataset Lists -->
  <DatasetListMIB1>DatasetList.xml</DatasetListMIB1>
  <DatasetListMIB2>DatasetList.xml</DatasetListMIB2>
  
  <!-- Paths -->
  <SwapFecPath>SwapFecs\</SwapFecPath>
  <ZdcPath>Zdc\</ZdcPath>
  <VirtualVehiclePath>prNumberConfiguration\</VirtualVehiclePath>
  <SfdTokenList>Sfd2TokenList.xml</SfdTokenList>
  <IvdHashPath>IvdHashList.xml</IvdHashPath>
</Config>
```

---

## Test-Entwicklung Workflows

### Workflow 1: Manuellen Test erstellen

```csharp
// 1. Neue .cs Datei in entsprechendem Ordner erstellen
// z.B. Production/MyCustomTest.cs

using DiagTestbase.TestLibMib;
using DiagTestbase.TestLibUds.Structures;
using DiagTestbase.TestLibMib.Templates;

namespace DiagTestbase.MibTestsGen3.Production
{
    public class MyCustomTest : TestMib
    {
        public MyCustomTest() : base(false, false, false) 
        {
            // Parameter registrieren
            ParameterHandler.AddParameter(
                ParameterHandler.DefaultParamResAndReqLogging, 
                ParameterType.Text, 
                true
            );
        }

        protected override TestCase CreateTestCase()
        {
            BaseTemplate tc = new BaseTemplate();
            tc.Id = "MY_CUSTOM_001";
            tc.Name = "My Custom Test";
            tc.Description = "Does something special";
            
            // Validity Filter (optional)
            tc.Validities.Set(
                VWSystemConfig.MIBGenerationType.MIB3, 
                ValidityKind.Require
            );
            
            // Testphasen
            tc.AddPreConditionMethod(tc.ExecuteBasePreConditions);
            tc.AddPreConditionMethod(ExecutePreconditions);
            tc.AddTestMethod(ExecuteTest);
            tc.AddPostConditionMethod(tc.ExecuteBasePostConditions);
            tc.AddTestIdLoggingMethod();
            
            return tc;
        }

        public bool? ExecutePreconditions()
        {
            // Deine Preconditions hier
            return true;
        }

        public bool? ExecuteTest()
        {
            bool isSuccess = true;
            
            // Dein Test hier
            // z.B.:
            Did did = MibDataRegistry.GetDataObjectDid(0xF186);
            TestStepDidRead read = new TestStepDidRead(did);
            isSuccess &= read.Execute(true);
            
            return isSuccess;
        }
    }
}

// 2. Manuell zu .csproj hinzufügen (oder Let Visual Studio do it)
// 3. Build & Deploy wie gewohnt
```

### Workflow 2: Bestehenden Test anpassen

**⚠️ ACHTUNG**: Generierte Tests NICHT direkt editieren!

**Wenn TCE-generiert**:
1. Test in TCE öffnen
2. Änderungen in TCE machen
3. Code neu generieren
4. Build

**Wenn manuell erstellt**:
1. .cs File direkt editieren
2. Build

---

## Naming Conventions

### Test-Dateinamen

```
DIAG_<Kategorie>_TI_TC_<Nummer>_<Beschreibung>_<Suffix>.cs

Beispiele:
DIAG_Ident_TI_TC_514_VW_ECU_Hardware_Number_T.cs
DIAG_Err_TI_TC_1980_Antenne_für_Satelliten_Tuner_Unterbrechung_CT.cs
DIAG_Rou_TI_TC_998_Start_Routine.cs
```

**Kategorien**:
- `Ident` - Identification
- `Err` - DTC / Error
- `Rou` - Routine
- `Stel` - Stellglied / Actuator
- `Mes` - Messwert / Measurement
- `Anp` - Anpassung / Adaption
- `KSD` - Kundendienst

**Suffix**:
- `T` - Test (standard)
- `CT` - Complex Test
- `CLT` - Complex Long Test
- `C` - Complex
- `L` - Long
- `LT` - Long Test

### Klassen-Namen

```csharp
// Entspricht Dateinamen (ohne .cs)
public class DIAG_Ident_TI_TC_514_VW_ECU_Hardware_Number_T : TestMib
```

### Test-IDs

```csharp
tc.Id = "DIAG_Ident_TI_TC_514";
// Format: DIAG_<Kategorie>_TI_TC_<Nummer>
```

---

## Best Practices

### 1. Immer BaseTemplate nutzen

```csharp
// ✅ RICHTIG
BaseTemplate tc = new BaseTemplate();
tc.AddPreConditionMethod(tc.ExecuteBasePreConditions);
tc.AddPostConditionMethod(tc.ExecuteBasePostConditions);

// ❌ FALSCH (kein Auto-Revert!)
TestCase tc = new TestCase();
```

### 2. User-Interaktion klar formulieren

```csharp
// ✅ RICHTIG
TestStepUserQuery query = new TestStepUserQuery(
    "Disconnect antenna for satellite tuner", 
    false  // nicht nur Info
);

// ❌ FALSCH (zu unklar)
TestStepUserQuery query = new TestStepUserQuery("Do something");
```

### 3. Logging strukturiert

```csharp
// ✅ RICHTIG
Logger.TestLogger.LogInformation("Step: Action [001]");
TestStepDidRead read = new TestStepDidRead(did);
isSuccess &= read.Execute(true);

// Grouped Logging
Logger.TestLogger.LogParagraph("Step: Precondition [001]", "Info", "");
```

### 4. Warte-Zeiten ausreichend

```csharp
// ✅ RICHTIG - ECU braucht Zeit
TestStepWait wait = new TestStepWait(11000);  // 11 Sekunden

// ❌ FALSCH - zu kurz
TestStepWait wait = new TestStepWait(100);  // 100ms nicht genug
```

### 5. DTC Status korrekt prüfen

```csharp
// ✅ RICHTIG - Status-Byte prüfen
TestStepDtcCheckStatus check = 
    new TestStepDtcCheckStatus(dtc, new DtcStatus(0x09));

// ❌ FALSCH - nur Presence prüfen (ungenau)
TestStepDtcCheckPresence check = new TestStepDtcCheckPresence(dtc, true);
```

### 6. Negative Tests explizit

```csharp
// ✅ RICHTIG - negative Response erwartet
TestStepRoutineRequestResults request = 
    new TestStepRoutineRequestResults(routine);
isSuccess &= request.Execute(false);  // false = NRC erwartet

// Dann NRC prüfen
if(request.Response is ResponseNegative negResp) {
    TestStepCompareNrc compareNrc = new TestStepCompareNrc(negResp, 0x24);
    isSuccess &= compareNrc.Execute();
}
```

### 7. isSuccess immer mit &= kombinieren

```csharp
// ✅ RICHTIG - Fehler propagieren
bool isSuccess = true;
isSuccess &= step1.Execute(true);
isSuccess &= step2.Execute(true);
isSuccess &= step3.Execute(true);
return isSuccess;  // false wenn EINER fehlschlägt

// ❌ FALSCH - nur letzter Wert zählt
bool isSuccess = step1.Execute(true);
isSuccess = step2.Execute(true);  // überschreibt!
```

---

## Häufige TestSteps

### Session Management
```csharp
TestStepSessionChange sessionEol = 
    new TestStepSessionChange(Session.Eol);
sessionEol.Execute(true);
```

### DID Operations
```csharp
// Read
Did did = MibDataRegistry.GetDataObjectDid(0xF191);
TestStepDidRead read = new TestStepDidRead(did);
read.Execute(true);

// Write
did.ByteValue = new byte[] { 0x01, 0x02 };
TestStepDidWrite write = new TestStepDidWrite(did);
write.Execute(true);
```

### DTC Operations
```csharp
// Read DTC
Dtc dtc = MibDataRegistry.GetDataObjectDtc(0x000009);
TestStepDtcReadInfo readDtc = 
    new TestStepDtcReadInfo(dtc, RequestReadDtcExtData.ExtDataRecNr.All);
readDtc.Execute(true);

// Check Status
TestStepDtcCheckStatus checkStatus = 
    new TestStepDtcCheckStatus(dtc, new DtcStatus(0x09));
checkStatus.Execute(true);

// Clear DTCs
TestStepDtcClear clear = new TestStepDtcClear();
clear.Execute(true);
```

### Routines
```csharp
Routine routine = MibDataRegistry.GetDataObjectRoutine(0x0252);

// Start
List<byte> options = new List<byte>{ 4, 0, 0 };
TestStepRoutineStart start = new TestStepRoutineStart(routine, options);
start.Execute(true);

// Request Results
TestStepRoutineRequestResults results = 
    new TestStepRoutineRequestResults(routine);
results.Execute(true);

// Stop
TestStepRoutineStop stop = new TestStepRoutineStop(routine);
stop.Execute(true);
```

### CAN Signals
```csharp
// Set CAN Signal
TestStepCanSignalSetValue setSignal = 
    new TestStepCanSignalSetValue(
        RestBusSimulationChannel.ICAN,  // Channel
        "ZAS_Kl_15",                     // Signal Name
        1                                 // Value (1 = ein)
    );
setSignal.Execute(true);
```

### User Interaction
```csharp
// Query
TestStepUserQuery query = 
    new TestStepUserQuery("Please disconnect USB", false);
query.Execute(true);

// Validation
TestStepUserQueryValidation validation = 
    new TestStepUserQueryValidation(did, "Check value matches label");
validation.Execute(true);
```

### Waits & Resets
```csharp
// Wait
TestStepWait wait = new TestStepWait(5000);  // 5 seconds
wait.Execute(true);

// Reset by Service
TestStepResetByService reset = 
    new TestStepResetByService(RequestEcuReset.ResetType.HardReset);
reset.Execute(true);

// Reset by Klemme
TestStepResetByKl klReset = 
    new TestStepResetByKl(Reset.Clamp15Automatic);
klReset.Execute(true);
```

---

## CI/CD Pipeline

### Azure DevOps Pipeline Files

```
PipelineBuildConfig/
├── mibtest-build-develop.yml        # Development Branch
├── mibtest-build-master.yml         # Release/Master Branch
├── mibtest-steps-template.yml       # Wiederverwendbare Steps
└── mibtest-env-variable-template.yml # Environment Variables
```

### Build Steps

1. **Restore NuGet Packages**
   ```yaml
   - task: NuGetCommand@2
     inputs:
       command: 'restore'
       restoreSolution: 'MibTestsGen3.sln'
       feedsToUse: 'config'
       nugetConfigPath: 'NuGet.config'
   ```

2. **Build Solution**
   ```yaml
   - task: VSBuild@1
     inputs:
       solution: 'MibTestsGen3.sln'
       configuration: 'Release'
       msbuildArgs: '/p:DITES_TESTSCRIPTS_PATH="$(Build.BinariesDirectory)"'
   ```

3. **Publish Artifacts**
   ```yaml
   - task: PublishBuildArtifacts@1
     inputs:
       pathtoPublish: '$(Build.BinariesDirectory)'
       artifactName: 'MibTests_Gen3'
   ```

### Environment Variable

```yaml
# In mibtest-env-variable-template.yml
variables:
  DITES_TESTSCRIPTS_PATH: '$(Build.BinariesDirectory)'
```

---

## Troubleshooting

### Problem: Test-DLL wird nicht in DITES angezeigt

**Lösung**:
1. Prüfe `DITES_TESTSCRIPTS_PATH` Environment Variable
2. Prüfe ob Post-Build Event ausgeführt wurde
3. Prüfe ob DLL existiert in: `%DITES_TESTSCRIPTS_PATH%\UDS\MIB_Tests_Gen3\<Kategorie>\`

### Problem: NuGet Packages nicht gefunden

**Lösung**:
1. Prüfe NuGet.config für Package Sources
2. Prüfe Zugriff auf interne NuGet Feeds
3. Restore manuell: `nuget restore MibTestsGen3.sln`

### Problem: ConfigMib.xml nicht gefunden

**Lösung**:
1. Prüfe Pfad in `Config.ReadFromFile()`
2. Stelle sicher dass XmlDefinitions-Projekt gebaut wurde
3. Prüfe ob XML als Content markiert ist (Copy to Output Directory)

### Problem: DID/DTC nicht in Registry

**Lösung**:
1. Prüfe ob entsprechende XML-Liste in ConfigMib.xml referenziert ist
2. Prüfe ob XML-File existiert
3. Prüfe XML Syntax (muss gegen Schema validieren)

### Problem: Test schlägt fehl mit Timeout

**Lösung**:
1. Erhöhe Warte-Zeiten (`TestStepWait`)
2. Prüfe ECU Status (ist ECU bereit?)
3. Prüfe CAN-Kommunikation
4. Prüfe Tester Present (läuft Auto-TP?)

---

## Verzeichnis-Mapping

### Von MibTest zu DITES Testscripts

```
MibTest/Identifications/bin/Release/
  MibTests_Identifications_Gen3.dll
  MibTests_Identifications_Gen3.xml
          ↓ (Post-Build Copy)
%DITES_TESTSCRIPTS_PATH%/UDS/MIB_Tests_Gen3/Identifications/
  MibTests_Identifications_Gen3.dll
  MibTests_Identifications_Gen3.xml
          ↓ (DITES lädt zur Laufzeit)
DITES Application
  ├─ Reflection: Assembly.LoadFrom(dll)
  ├─ Find ITest implementations
  ├─ CreateInstance()
  └─ ExecuteTest(IFramework)
```

---

## Zusammenfassung

### MibTest ist:
**Test-Implementierungs-Repository** - Konkrete Tests für MIB3  
**Code-Generierungs-Target** - TCE generiert Code hierhin  
**Build-Artefakt-Quelle** - Produziert DLLs für DITES  
**Abhängig von Diag-Testbase** - Nutzt Framework via NuGet  
**Integration mit DITES** - DLLs werden von DITES geladen  

---

*Ende der MibTest Dokumentation - Teil 2*