# Strato-deploy stappenplan — chopchopamsterdam.com → Lucky Duck

**Doel:** chopchopamsterdam.com schakelt op donderdag 28 mei 2026 om 21:00 over van Squarespace naar de Vercel-gehoste redirect-pagina.

**Strategie:** Vercel host de page, Strato wijst alleen het domein door. Geen file uploads naar Strato webspace nodig.

---

## Hosting

| | |
|---|---|
| Live URL | https://lucky-duck-chopchop-redirect.vercel.app |
| Vercel project | `lucky-duck-chopchop-redirect` |
| GitHub repo | https://github.com/TomTeurlings1106/lucky-duck-chopchop-redirect |

Eerst even openen in browser om te checken dat 'm goed staat.

---

## STAP 1 — Vooraf bij Vercel (5 min, kan vandaag of morgen overdag)

1. https://vercel.com/toms-projects-c66df615/lucky-duck-chopchop-redirect/settings/domains
2. Klik **Add Domain**
3. Voer in: `chopchopamsterdam.com` → Add
4. Voer ook in: `www.chopchopamsterdam.com` → Add (zet `www` als redirect naar apex of andersom)
5. Vercel toont nu **2 vereiste DNS-records** — noteer ze:
   - Voor apex (`chopchopamsterdam.com`): **A-record** naar een Vercel IP (meestal `76.76.21.21`)
   - Voor www: **CNAME-record** naar `cname.vercel-dns.com`
6. Status zal "Invalid Configuration" zijn — dat is OK, fixen we in stap 3

---

## STAP 2 — Vooraf bij Strato (5 min, **minstens 4u** vóór de flip)

Doel: TTL verlagen zodat de DNS-flip straks snel propageert.

1. Login Strato → **Domain** → `chopchopamsterdam.com` → **DNS-instellingen** (of "Nameserver/DNS-Verwaltung")
2. **Belangrijk: noteer of screenshot de huidige A/CNAME-records van Squarespace** (rollback)
3. Wijzig de TTL van alle records naar **300 seconden** (5 min)
4. Opslaan
5. Wacht minstens 4u — pas dan zijn oude TTL-cached records overal verlopen

---

## STAP 3 — De flip (donderdag 28 mei, **20:55**)

1. Login Strato → DNS-instellingen `chopchopamsterdam.com`
2. **Verwijder** de huidige A-record(s) die naar Squarespace wijzen
3. **Voeg toe** A-record:
   - Host: `@` (of leeg, betekent apex)
   - Type: `A`
   - Waarde: `76.76.21.21` (Vercel IP — gebruik exact wat Vercel in stap 1 toonde)
   - TTL: 300
4. **Verwijder/wijzig** de www-record:
   - Host: `www`
   - Type: `CNAME`
   - Waarde: `cname.vercel-dns.com`
   - TTL: 300
5. Opslaan

---

## STAP 4 — Verificatie (21:00–21:10)

1. Open **incognito-venster** (cache uit) → bezoek `https://chopchopamsterdam.com`
2. Moet de redirect-pagina tonen met CTA "Visit Lucky Duck"
3. Bezoek ook `https://www.chopchopamsterdam.com` — zelfde resultaat
4. Test ook op mobiel (4G, niet WiFi — andere DNS)
5. Klik de CTA → moet naar `luckyduckamsterdam.com` gaan

Bij DNS-vertraging:
- Forceer DNS-flush op Mac: `sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder`
- Check propagation: https://dnschecker.org/#A/chopchopamsterdam.com

---

## ROLLBACK (als er iets fout gaat)

1. Login Strato → DNS-instellingen
2. Zet de oude Squarespace A/CNAME records terug (uit stap 2)
3. Binnen 5 min weer terug op Squarespace

---

## Tekst aanpassen later

Tekst zit in `index.html` — wijzigen, committen + pushen naar GitHub. Vercel deployt automatisch:

```bash
cd chopchop-redirect
# edit index.html
git add index.html
git commit -m "[LCK-099] Update redirect-tekst"
git push
```

Binnen ~30 sec is de nieuwe versie live op `chopchopamsterdam.com`.
