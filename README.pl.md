# Security Posture Review — przed i po hardeningu

🇬🇧 [English version](README.md)

Celowo wprowadziłem błędy konfiguracji we własnym labie Azure, a potem zrobiłem audyt przy użyciu **Microsoft Defender for Cloud**, **Azure Policy** i **przeglądu ręcznego**. Każde znalezisko naprawiłem i określiłem, jak zapobiec jego powtórce.

📄 **Pełny raport:** [raport_PL.pdf](raport_PL.pdf) · [report_EN.pdf](report_EN.pdf)

## Wyniki

| Wskaźnik | Przed sabotażem | Po sabotażu | Po naprawie* |
|---|---|---|---|
| Secure Score | 100% | 9% | 69% |
| MCSB — zaliczone kontrole | 57/63 | 46/63 | 55/63 |
| Rekomendacje Unhealthy — Storage + Key Vault | — | 9 | 3 |

\* Pomiar wykonany po usunięciu VM (ze względu na koszty), więc część wzrostu wynika z usunięcia maszyny. Pozostałe 3 rekomendacje nie wynikają z sabotażu (private link / reguły VNet).

**Pokrycie 8 wprowadzonych błędów:** Defender for Cloud **6/8**, Azure Policy **7/8**, wyłącznie przegląd ręczny **1/8**.

## Ustalenia

| # | Znalezisko | Severity | Wykryte przez |
|---|---|---|---|
| 1 | Anonimowy dostęp do kontenera | Medium | Defender + Policy |
| 2 | Brak wymuszonego HTTPS (secure transfer) | High | Defender + Policy |
| 3 | Storage dostępny ze wszystkich sieci | Medium | Defender + Policy |
| 4 | Brak logów diagnostycznych Key Vault | Low | Defender + Policy |
| 5 | Brak purge protection w Key Vault | Medium | Defender + Policy |
| 6 | Key Vault dostępny ze wszystkich sieci | Medium | Defender + Policy |
| 7 | SSH (22) otwarty na internet | High (ocena własna) | **tylko Policy** |
| 8 | Rola Owner (nadawanie wszystkich ról) dla dewelopera | High (ocena własna) | **tylko przegląd ręczny** |

## Najważniejsze wnioski

1. **Brak rekomendacji ≠ brak problemu.** Defender nie wykrył otwartego portu SSH (zgłosiła go tylko Policy) ani nadmiarowych uprawnień dewelopera — narzędzie widzi konfigurację, a nie kontekst biznesowy.
2. **Audit wykrywa, Deny zapobiega.** Domyślna inicjatywa ASC tylko audytuje. Z efektem Deny 5 z 8 błędów nie dałoby się utworzyć; z DeployIfNotExists 6 z 8 byłoby zablokowanych lub naprawionych automatycznie.
3. **Otwarte porty zarządzania są atakowane w ciągu godzin.** 19 nieudanych logowań SSH z 14 adresów IP w ok. 4 h na nieużywanej VM. Zatrzymało je logowanie kluczem — kontrola kompensująca, nie naprawa.

## Zakres

- Subskrypcja pay-as-you-go, jedna grupa zasobów, West Europe
- Konto Storage, Key Vault, VM Linux (D2 v3) z NSG i publicznym IP, Log Analytics, konto testowe w Entra ID
- Defender for Cloud w wersji darmowej (Foundational CSPM), Azure Policy (ASC Default / MCSB), przegląd ręczny IAM, reguł NSG i logów SSH
- Listopad 2026

## Dowody

Zanonimizowane zrzuty są w folderze [evidence/](evidence/). Identyfikatory (adresy IP, identyfikatory subskrypcji, nazwy kont, dane osobowe) zostały celowo usunięte — puste pola wynikają z anonimizacji, a nie z konfiguracji. Eksporty CSV nie są publikowane, bo zawierają identyfikatory.
