# TS 240 – Auknar kortafærslur
### Sannprófunarreglur fyrir kortafærslukvittanir

TS 240 er íslenskur staðall (Staðlaráð Íslands) fyrir **kortafærslukvittanir** — rafræna skráningu á
kortagreiðslu, sett fram sem venjulegur rafrænn reikningur eða kreditreikningur. Hann byggir á evrópska
rafræna reikningsstaðlinum **EN 16931** og **Peppol BIS Billing 3.0**. Þessi mappa geymir reglurnar sem
sannreyna hvort slík kvittun sé rétt og fullgerð, ásamt tilbúnum dæmum. Sjálfar viðskiptareglurnar eru
raktar sem Staðlaráðs-mál #4; þessi mappa er vélræn útfærsla þeirra.

Kvittun er auðkennd með:
- **ProfileID** `urn:fdc:stadlar.is:ts240:1.0`
- **CustomizationID** `urn:fdc:stadlar.is:kort:1.0`

## Hvernig á að nota

Reglurnar koma í tveimur skrám (ISO Schematron þýtt í **XSLT 2.0**):

| Skrá | Hvað hún sannprófar |
|---|---|
| `EN-validate-receipt.xsl` | Evrópska staðalinn (EN 16931): samtölur, VSK-útreikninga, gilda kóða, skyldureiti. |
| `STADLAR-validate-receipt.xsl` | Íslensku (Staðlaráðs) reglurnar og þær sem eru sértækar fyrir kortafærslur. |

Sannprófaðu kvittun með því að keyra **báðar skrárnar, í röð — EN fyrst, svo STADLAR** — með hvaða
XSLT 2.0 vél sem er (t.d. Saxon-HE). Hver keyrsla skilar hefðbundinni **SVRL** skýrslu sem telur upp það
sem stóðst ekki. Niðurstöður eru annaðhvort:
- **Villur** — kvittunin er ógild og þarf að leiðrétta.
- **Viðvaranir** — kvittunin er samþykkt en yfirfara ætti eitthvað.

**Einnig í möppunni:**
- `examples/` — átta tilbúnar kvittanir, ein fyrir hverja sviðsmynd, sem allar standast án villu:
  - `…-domestic` — íslenskur seljandi, staðlað 24 % VSK.
  - `…-domestic-novat` — íslenskur seljandi þar sem VSK er ekki sundurliðaður (t.d. óendurkræf
    veitingakaup).
  - `…-multiline` — margar línur með blönduðum VSK-þrepum.
  - `…-ecommerce` — netverslun / kort ekki til staðar.
  - `…-foreign-merchant` — erlendur seljandi, upphaflega upphæðin skráð í BT-25.
  - `…-minimal` / `…-minimal-foreign` — minnsta gilda innlenda / erlenda kvittunin.
  - `…-creditnote-return` — endurgreiðsla á kort (kreditreikningur).
- `ts240-schematron-adaptations.md` — frávikaskrá: nákvæm útlistun reglu fyrir reglu að baki „Útfærslu“
  hér að neðan (á ensku, fyrir þróunaraðila).

## Opin atriði

Þessi atriði eru enn opin í
[Staðlaráðs-málaskránni](https://github.com/stadlar/TS-240-Kortafaerslur/issues) og hafa áhrif á
sannprófun:

- **Greiðslumátakóði** — kvittanirnar nota `54` (kreditkort); eldri tafla FJS notaði `48`
  (greiðslukort). Velja þarf annað gildið, eða leyfa bæði. *(Staðlaráðs-mál #5.)*
- **Möskun kortanúmers** — staðfesta þarf nákvæmt möskunarform (t.d. `542418******1234`). *(Mál #10.)*
- **Nafn og kennitala korthafa** — staðallinn hefur enn engan reit fyrir þessi gildi. Tillaga okkar:
  setja **nafn** korthafa í nafnreit kortareiknings (BT-88) og **kennitölu** korthafa í hlutaauðkenni
  reiknings með kóðanum `ARR` (UNTDID 1153 — kennitala einstaklings). Yfirlestur FJS leggur til
  BT-82 / BT-83, sem eru aðrir reitir í EN 16931, svo staðfesta þarf nákvæma reitanúmerun. *(Mál #10.)*
- **ARN (acquirer reference number — tilvísunarnúmer færsluhirðis)** — á sér engan ákveðinn reit í
  prófílnum enn. *(Mál #12.)*

## Útfærsla

Kortafærslukvittun er áfram **fullgildur EN 16931 reikningur eða kreditreikningur**; eftirfarandi
athuganir bætast ofan á. Þeim er skipt eftir því hvernig þær tengjast TS 240 forskriftinni.

### Fylgir forskriftinni
Framfylgt nákvæmlega eins og TS 240 kveður á um:
- Skjalið er kortafærslukvittun (**tegund 461**).
- Það er **greitt með korti** og **greiðsluupphæðin er núll** (þegar uppgert).
- **ISK-upphæðir bera enga aukastafi** (venjulegur tugatölureitur, svo `1000.00` er samþykkt).
- Viðhengd kvittanamynd/PDF helst innan **stærðar- og skráarnafnsmarka**.
- Fyrir **erlendan seljanda** ætti að skrá upphaflega upphæð og gjaldmiðil.
- Allar hefðbundnar reikningsathuganir gilda — samtölur stemma, VSK er rétt reiknaður, kóðar eru gildir
  og skylduaðilar til staðar.

### Viðbótarkröfur
Aðeins strangari en orðalag draganna, til að halda gögnunum hreinum og samræmdum:
- **Gjaldmiðillinn er alltaf ISK** (drögin segja „usually“).
- **Tilvísun sendanda (fjártækniaðila) verður að vera til staðar** — frjáls texti sem geymir nafn
  fjártækniaðilans (t.d. „Fjártæknilausnin“); ef hana vantar er það tilkynnt skýrt.
- **Hvernig kortið var notað** (*POS entry mode*) er valfrjálst, en ef það er tilgreint ætti það að vera
  eitt af stuttum stöðluðum lista (`MANUAL`, `MAGSTRIPE`, `CHIP`, `CONTACTLESS`, `ECOMMERCE`, `MOTO`,
  `STORED`); annað gefur viðvörun.

### Frávik frá drögunum
Þar sem drög v0.4 stangast á við evrópska staðalinn fylgja reglurnar staðlinum (og það er tilkynnt
Staðlaráði):
- **Greiðslumátakóðinn er 54**, ekki 58 (58 er ekki kortakóði og fellur á staðalreglu).
- **Lægra VSK-þrepið er staðalflokkurinn með 11 %**, ekki sérstakur „AA“-kóði (sem evrópski staðallinn
  leyfir ekki).
- Þegar seljandi hefur **ekkert VSK-númer** — erlendur seljandi, eða innlend kvittun sem sundurliðar ekki
  VSK — er notaður **flokkurinn „utan gildissviðs VSK“**. (Þetta svarar Staðlaráðs-máli #8 — kortaveitur
  gefa yfirleitt engin VSK-gögn.)
- Kortafærslu-skjalategundin **461 er samþykkt**.
- **Möskunarathugun kortanúmers samþykkir tákn-maskað númer.**
- Fáeinar **íslenskar reikningsathuganir sem eiga ekki við kortakvittanir** (götuheiti/póstnúmer,
  gjalddagi) eru gerðar óvirkar.

### Tillögur fyrir v0.5 — óútfært
Ekki útfært; tillögur til Staðlaráðs / FJS:
- **Netfang fjártækniaðila** — ef geyma þarf það á það heima í netfangsreit seljanda;
  tilvísunarreiturinn helst frjáls texti (nafn fjártækniaðilans), hann er **ekki** felldur niður.
- **Formfesta reitinn „hvernig kortið var notað" (POS entry mode) og gildalistann** í forskriftinni (í
  dag er þetta valfrjáls athugun sem gefur aðeins viðvörun).
- **Bæta við reitum fyrir nafn og kennitölu korthafa** (sjá tillögu í „Opin atriði“).
- **Bæta flokknum „utan gildissviðs VSK“ formlega** við lista forskriftarinnar.
- **Smávægilegar textaleiðréttingar** í forskriftinni:
  - **Kóðakerfi** — §4.1 tilgreinir kerfi íslenskrar kennitölu sem `IS:KT`; á að vera `0196`
    (EAS/Peppol). §5.1 merkir BT-18-1 / BT-128-1 sem „ISO code list“; þau eru í raun **UNTDID 1153**.
  - **Ófullgerð reitlýsing** — §3.2 BT-49 (rafrænt vistfang kaupanda) segir aðeins „Endpoint?“; þarf að
    klára.
  - **Innsláttarvilla í einingalista** — §5.1 BT-130 segir að einingalistinn sé „recommendation 20 and
    12“; á að vera **20 og 21**.

## Reglur í hnotskurn

Sértækar kortafærsluathuganir (ofan á áðurgildandi EN 16931 / Peppol reglur):

| Regla | Athugar að… | Flokkur | Alvarleiki |
|---|---|---|---|
| TS240-R-001 | skjalategundin sé 461 (kortafærslukvittun) | Fylgir forskrift | Villa |
| TS240-R-002 | greiðslukóðinn sé 54 (kreditkort) | Frávik | Villa |
| TS240-R-003 | greiðsluupphæðin sé núll | Fylgir forskrift | Villa |
| TS240-R-004 | ISK-upphæðir beri enga aukastafi | Fylgir forskrift | Villa |
| TS240-R-005 | viðhengt skjal sé í mesta lagi 5 MB | Fylgir forskrift | Villa |
| TS240-R-006 | skráarnafn viðhengis sé í mesta lagi 255 stafir | Fylgir forskrift | Villa |
| TS240-R-007 | erlendur seljandi skrái upphaflega upphæð og gjaldmiðil | Fylgir forskrift | Viðvörun |
| TS240-R-008 | seljandi án VSK-númers noti flokkinn „utan gildissviðs VSK“ | Frávik | Villa |
| TS240-R-009 | gjaldmiðill skjalsins sé ISK | Viðbótarkrafa | Villa |
| TS240-R-010 | tilvísun sendanda (fjártækniaðila) sé til staðar | Viðbótarkrafa | Villa |
| TS240-R-011 | POS entry mode (hvernig kortið var notað), ef tilgreint, sé eitt af stöðluðu gildunum | Viðbótarkrafa | Viðvörun |

## Þekja forskriftar

Þetta varpar hverri kröfu í forskrift TS 240 (§5.1) á regluna sem uppfyllir hana.
**Fyrirliggjandi** = áðurgildandi EN 16931 / Peppol regla; **Ný** = regla búin til fyrir TS 240;
**Viðbót** = regla sem við bættum við og er ekki enn í forskriftinni (lögð til upptöku).

| Krafa (BT) | Uppfyllt af | Tegund |
|---|---|---|
| BT-2 — Útgáfudagur | BR-02 (dagsetning skylda; „= færsludagsetning“ er gagnakrafa) | Fyrirliggjandi |
| BT-3 — Tegundarkóði 461 | TS240-R-001 + BR-CL-01 | Ný |
| BT-5 — Gjaldmiðill (ISK) | TS240-R-009 + BR-CL-04 | Ný |
| BT-18-1 — Kóðakerfi hlutaauðkennis | BR-CL-07 (UNTDID 1153) | Fyrirliggjandi |
| BT-23 — Rekstrarferli | PEPPOL-EN16931-R001 / R007 | Fyrirliggjandi |
| BT-24 — Forskriftarauðkenni | PEPPOL-EN16931-R004 | Fyrirliggjandi |
| BT-30 — Lögaðilaauðkenni seljanda | IS-R-002 + BR-CO-26 | Fyrirliggjandi |
| BT-31 — VSK-númer seljanda (IS-forskeyti) | BR-CO-09 | Fyrirliggjandi |
| BT-34 — Rafrænt vistfang seljanda | PEPPOL-EN16931-R020 + BR-62 | Fyrirliggjandi |
| BT-34-1 — Kóðakerfi vistfangs seljanda | BR-CL-25 (EAS) | Fyrirliggjandi |
| BT-40 — Land seljanda | BR-09 + BR-CL-14 | Fyrirliggjandi |
| BT-44 — Nafn kaupanda | BR-07 („korthafi“ er gagnakrafa) | Fyrirliggjandi |
| BT-47 — Lögaðilaauðkenni kaupanda | IS-R-004 | Fyrirliggjandi |
| BT-49 — Rafrænt vistfang kaupanda | PEPPOL-EN16931-R010 + BR-63 | Fyrirliggjandi |
| BT-55 — Land kaupanda | BR-11 + BR-CL-14 | Fyrirliggjandi |
| BT-81 — Greiðslumátakóði | TS240-R-002 (54) | Ný |
| BT-87 — Kortanúmer (möskun) | BR-51 (aðlagað) | Fyrirliggjandi |
| BG-22 — Heildartölur | BR-CO-10/13/14/15/16 + BR-DEC-* | Fyrirliggjandi |
| BT-106/109/110/112/113 — Fjárhæðir (ISK, engir aukastafir) | TS240-R-004 + BR-CO-* / BR-DEC-* | Ný |
| BT-115 — Greiðsluupphæð = 0 | TS240-R-003 + BR-CO-16 + BR-16 | Ný |
| BT-116 — VSK-grunnur flokks | BR-S-08 + BR-DEC-19 | Fyrirliggjandi |
| BT-125 — Viðhengi ≤ 5 MB | TS240-R-005 | Ný |
| BT-125-1 — MIME-kóði viðhengis | BR-CL-24 | Fyrirliggjandi |
| BT-125-2 — Skráarnafn viðhengis ≤ 255 | TS240-R-006 | Ný |
| BT-128 — Hlutaauðkenni línu | PEPPOL-EN16931-R101 | Fyrirliggjandi |
| BT-128-1 — Kóðakerfi hlutaauðkennis línu | BR-CL-07 | Fyrirliggjandi |
| BT-130 — Mælieining | BR-CL-23 (Rec 20/21) | Fyrirliggjandi |
| BT-131 — Nettóupphæð línu | BR-DEC-23 | Fyrirliggjandi |
| BT-133 — Bókhaldstilvísun línu | áðurgildandi línuregla; engin gildisathugun | Fyrirliggjandi |
| BG-30 — VSK-upplýsingar línu | BR-CO-04 + BR-S-* | Fyrirliggjandi |
| BT-153 — Heiti vöru | BR-25 | Fyrirliggjandi |
| BT-13 — Tilvísun fjártækniaðila | TS240-R-010 | Viðbót |
| BT-25 — Erlend upphæð upprunans | TS240-R-007 (viðvörun) | Viðbót |
| VSK-flokkur O (seljandi án VSK-númers) | TS240-R-008 | Viðbót |
| BT-82 — POS entry mode (hvernig kortið var notað) | TS240-R-011 (viðvörun) | Viðbót |

---

*Byggt á Peppol BIS Billing 3.0 / EN 16931. Nákvæma útlistun reglu fyrir reglu má sjá í
[ts240-schematron-adaptations.md](./ts240-schematron-adaptations.md).*
