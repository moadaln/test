# MibTest Repository - Test Implementation Documentation

## Überblick

**MibTest** ist das Test-Implementierungs-Repository, das **konkrete UDS-Diagnosetests** für **Volkswagen/Audi MIB (Media Information Bus)** Systeme enthält. Es nutzt das **Diag-Testbase Framework** und wird von **DITES** zur Laufzeit ausgeführt.

### Hauptzweck
- Implementierung konkreter Test Cases basierend auf VW-Spezifikationen
- Automatisierte UDS-Diagnosetests für MIB3 Infotainment-Systeme
- End-of-Line (EOL) Testing und Produktionsabläufe
- Supplier-Testing gemäß UDS-Prüfvorschriften

### Entwickelt von
**Expleo Germany GmbH** - Automotive Testing Division

---

## Build Status

| Branch | Status |
|--------|--------|
| **Develop** | [![Build](https://dev.azure.com/expleo-diagnostics/Diagnose-SB-TA/_apis/build/status/MIB-Test%20Development?branchName=develop)](https://dev.azure.com/expleo-diagnostics/Diagnose-SB-TA/_build/latest?definitionId=13&branchName=develop) |
| **Master** | [![Build](https://dev.azure.com/expleo-diagnostics/Diagnose-SB-TA/_apis/build/status/MIB-Test%20Release?branchName=master)](https://dev.azure.com/expleo-diagnostics/Diagnose-SB-TA/_build/latest?definitionId=12&branchName=master) |

---

## Technologie-Stack

- **Sprache**: C# (.NET Framework 4.8)
- **Dependencies**: 
  - **Diag-Testbase** (via NuGet) - Test Framework
  - **DITES** (via NuGet) - Framework Interface
- **Build-System**: MSBuild / Visual Studio Solution
- **CI/CD**: Azure DevOps Pipelines
- **Code Generation**: TestCaseEditor (TCE)

---

## Repository-Struktur

```
MibTest/
├── Actuators/              # Stellglied-Tests (IoControl)
├── Adaptions/              # Adaptionswert-Tests
├── Coding/                 # Codierungs-Tests
├── DataSets/               # Dataset-Tests
├── Dtcs/                   # Fehlerspeicher (DTC) Tests
├── Identifications/        # Identifikationsdaten-Tests
├── KSD/                    # Kundendienst-spezifische Tests
├── Measurements/           # Messwertblock-Tests
├── MetaData/               # Test Attributes & Metadata
├── Production/             # Produktions-/EOL-Tests
├── Routines/               # Routine-Tests
├── XmlDefinitions/         # XML-Konfigurationen & Definitionen
│   ├── ConfigMib.xml       # Haupt-Konfiguration
│   ├── DidList_*.xml       # DID Definitionen
│   ├── DtcList_MIB.xml     # DTC Definitionen
│   ├── RoutineList_MIB.xml # Routine Definitionen
│   ├── Datasets/           # Dataset XML Files
│   ├── prNumberConfiguration/ # PR-Nummern Konfigurationen
│   └── Zdc/                # ZDC Files (Steuergeräte-IDs)
└── PipelineBuildConfig/    # Azure DevOps Pipelines
```

---

## Test-Kategorien

### 1. **Identifications** (Identifikationsdaten)
Tests für Steuergeräte-Identifikationsinformationen.

**Beispiele**:
- Hardware-Nummer (0xF191)
- Software-Version (0xF189)
- Seriennummer (0xF18C)
- VIN (0xF190)
- Slave-Component-Listen

**Typischer Ablauf**:
1. DID auslesen
2. User-Validierung (Wert gegen Label prüfen)

### 2. **Dtcs** (Diagnostic Trouble Codes)
Tests für Fehlerspeicher-Funktionalität.

**Test-Typen**:
- **Storing Check**: Fehler wird unter Bedingungen gespeichert
- **Precondition Check**: Fehler nur bei bestimmten Vorbedingungen
- **Passive Only**: Fehler wird nur passiv gesetzt

**Typischer Ablauf**:
1. Fehler provozieren (Hardware trennen, CAN-Signale setzen)
2. DTC Status prüfen (0x09 = testFailed | confirmedDTC)
3. Fehler beheben
4. DTC Status prüfen (0x08 = testFailedThisOperationCycle)
5. DTC löschen

### 3. **Routines** (Service 0x31)
Tests für ECU-Routinen.

**Test-Typen**:
- Start Routine (0x01)
- Stop Routine (0x02)
- Request Results (0x03)
- VW-spezifische Routinen (Checksumme, Kalibrierung, etc.)

**Typischer Ablauf**:
1. Routine-Status prüfen (MWB 0x0102)
2. Routine starten mit ControlOptions
3. Warten auf Completion
4. Results prüfen

### 4. **Actuators** (Stellglieder / IoControl)
Tests für Stellglied-Aktivierung (Service 0x2F).

**Test-Typen**:
- Short-Term (sofort beendet)
- Long-Term (läuft mehrere Sekunden)
- Negative Tests (falsche Bedingungen)

### 5. **Coding** (Codierung)
Tests für Codierungs-Schreib-/Lesevorgänge.

**Ablauf**:
1. Original-Codierung sichern
2. Neue Codierung schreiben
3. Validieren
4. Original zurückschreiben

### 6. **Measurements** (Messwertblöcke)
Tests für Diagnosemesswerte.

**Beispiele**:
- Spannungen
- Temperaturen
- Status-Flags
- Netzwerk-Informationen

### 7. **Production** (Produktionstests)
End-of-Line und Produktionsabläufe.

**Wichtige Tests**:
- `Unit_Parametrieren` - ECU parametrieren
- `Produktionsablauf_MIB3_CT` - Kompletter Level-0-Test
- `Werkseinstellungen` - Factory Reset
- SFD (Service Function Datei) Freischaltung

### 8. **Adaptions** (Adaptionswerte)
Tests für fahrzeugspezifische Anpassungen.

### 9. **DataSets** (Datasets)
Tests für Dataset-Upload/Download.

### 10. **KSD** (Kundendienst)
Spezielle Tests für After-Sales / Service.

---

## TestMib - Basis-Klasse

Alle Tests erben von `TestMib` (aus Diag-Testbase):

```csharp
public class MyTest : TestMib
{
    public MyTest() 
        : base(useCanOe, useRelayBox, usePowerSupply)
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
        tc.Id = "TC_12345";
        tc.Name = "My Test";
        
        tc.AddPreConditionMethod(tc.ExecuteBasePreConditions);
        tc.AddPreConditionMethod(ExecutePreconditions);
        tc.AddTestMethod(ExecuteTest);
        tc.AddPostConditionMethod(tc.ExecuteBasePostConditions);
        tc.AddPostConditionMethod(ExecutePostconditions);
        tc.AddTestIdLoggingMethod();
        
        return tc;
    }
    
    public bool? ExecutePreconditions() { /* ... */ }
    public bool? ExecuteTest() { /* ... */ }
    public bool? ExecutePostconditions() { /* ... */ }
}
```

---

## Test-Implementierungs-Beispiele

### Beispiel 1: Identification Test

**File**: `DIAG_Ident_TI_TC_514_VW_ECU_Hardware_Number_T.cs`

```csharp
public class DIAG_Ident_TI_TC_514_VW_ECU_Hardware_Number_T : TestMib
{
    public DIAG_Ident_TI_TC_514_VW_ECU_Hardware_Number_T()
        : base(false, false, false) // kein CANoe, RelayBox, PowerSupply
    {
        ParameterHandler.AddParameter(
            ParameterHandler.DefaultParamResAndReqLogging, 
            ParameterType.Text, 
            true
        );
    }

    protected override TestCase CreateTestCase()
    {
        BaseTemplate tc = new BaseTemplate();
        tc.Id = "DIAG_Ident_TI_TC_514";
        tc.Name = "VW_ECU_Hardware_Number_T";
        
        tc.AddPreConditionMethod(tc.ExecuteBasePreConditions);
        tc.AddTestMethod(this.ExecuteTest);
        tc.AddPostConditionMethod(tc.ExecuteBasePostConditions);
        tc.AddTestIdLoggingMethod();
        
        return tc;
    }

    public bool? ExecuteTest()
    {
        bool isSuccess = true;
        
        // DID 0xF191 (VW ECU Hardware Number) auslesen
        Did hwNumber = MibDataRegistry.GetDataObjectDid(0xF191);
        TestStepDidRead readStep = new TestStepDidRead(hwNumber);
        isSuccess &= readStep.Execute(true);
        
        // User-Validierung: Wert gegen Label prüfen
        TestStepUserQueryValidation validation = 
            new TestStepUserQueryValidation(
                hwNumber, 
                "Positive response with 11 byte ASCII string. " +
                "Returned value matches with value from label on unit"
            );
        isSuccess &= validation.Execute(true);
        
        return isSuccess;
    }
}
```

**Ablauf**:
1. Hardware-Nummer vom ECU lesen
2. Tester validiert Wert gegen Label am Gerät
3. Test erfolgreich wenn Werte