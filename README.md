# CODESYS CIP-pesuaseman PLC-ohjaus (IEC 61131-3 ST)

Teollisuusautomaation ohjelmistoprojekti, jossa **CIP-pesuaseman (Clean-in-Place)** prosessilogiikka ja turvalukitukset on toteutettu **CODESYS V3.5 SP22** -ympäristössä puhtaalla **IEC 61131-3 Structured Text (ST)** -kielellä.

Projekti toimii suorana PLC-vastinparina erilliselle [plc-iot-simulaattori](https://github.com/Laolli735/plc-iot-simulaattori) -repositoriolle: molemmat toteuttavat täsmälleen saman ISA-88-tilakoneen ja **OPC UA** -rajapinnan, mikä mahdollistaa samojen **Robot Framework** -tehdastestien (FAT) ajamisen sekä virtuaalista Python-simulaattoria että aitoa CODESYS-ohjainta vasten.

## Prosessilogiikka ja tilakone (`PLC_PRG`)

Ohjelma suorittaa determinististä tilakonetta ja 500 ms prosessimallinnusta (`TON`-ajastimet):

```text
[ 0: IDLE (Valmiustila) ]
       │  (Cmd_Start = TRUE)
       ▼
[ 1: PRE_RINSE (Esihuuhtelu) ] ──► Pumppu ON, lämmitys -> 40 °C (T#3S)
       │
       ▼
[ 2: CAUSTIC_WASH (Lipeäpesu) ] ──► Lipeäventtiili ON, lämmitys -> 75 °C, johtokyky -> 45 mS/cm (T#4S)
       │
       ▼
[ 3: FINAL_RINSE (Loppuhuuhtelu) ] ──► Lipeäventtiili OFF, huuhtelu kunnes johtokyky <= 2.0 mS/cm
       │
       └───────────────────────────► Automaattinen paluu tilaan 0 (IDLE)

* Turvalukitus: Cmd_EStop = TRUE pysäyttää pumpun ja sulkee lipeäventtiilin välittömästi tilaan [ 99: ALARM / E-STOP ].

OPC UA -rajapinta (Symbol Configuration)Muuttujat julkaistaan ulkoisille SCADA-, HMI- ja testausjärjestelmille CODESYSin sisäisen OPC UA -palvelimen kautta (Application.PLC_PRG):Symboli (Tagi)IEC-tietotyyppiSuuntaKuvausCmd_StartBOOLRead / WritePesuohjelman käynnistyskomentoCmd_EStopBOOLRead / WriteHätäpysäytys- ja turvalukitustietoStep_IDINTRead-onlyAktiivisen tilakonevaiheen numero (0, 1, 2, 3, 99)Step_NameSTRING(40)Read-onlyAktiivisen vaiheen selkokielinen nimiTemperature_CREALRead-onlyPesuliuoksen lämpötila (°C)Conductivity_mScmREALRead-onlyPaluulinjan sähkönjohtokyky (mS/cm)Pump_RunningBOOLRead-onlyKiertovesipumpun käyntitietoCaustic_Valve_OpenBOOLRead-onlyLipeäannosteluventtiilin ohjaustilaRepositorion rakenne ja käyttöönottocip-ohjaus.project – Varsinainen CODESYS V3.5 SP22 -projektitiedosto.PLC_PRG.st / *.xml – Tekstimuotoinen IEC 61131-3 ST -lähdekoodi ja PLCopenXML-vienti suoraan selaimessa tarkastelua varten.Käyttö CODESYS-ympäristössäAvaa .project-tiedosto CODESYS V3.5 -kehitysympäristössä (kohdelaite: CODESYS Control Win V3 x64).Varmista, että Symbol Configuration on käännetty (Build -> Generate Code / F11), jotta OPC UA -tagit aktivoituvat.Valitse Online -> Simulation (tai yhdistä paikalliseen Control Win -ajoympäristöön), kirjaudu sisään (Login) ja käynnistä ohjelma (Start / F5).