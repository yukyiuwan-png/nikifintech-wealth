# HK Client Wealth Check · 客戶理財診斷

香港客戶理財診斷的單頁工具：在瀏覽器內估算現金流、薪俸稅、強積金供款、債務、應急現金、人壽保額經驗法則、退休與理財目標，並可列印報告。財務數字只留在本機。帳號電郵、最近登入時間及功能使用紀錄存入 Firestore（香港，asia-east2），用以編製使用統計，並供登入。Firebase Authentication 是 Google 的全球服務，登入資料仍可能在香港以外處理。本工具只供教育參考，不是財務、保險或強積金意見，亦未加入付款或訂閱。

A single-page tool for a Hong Kong client wealth check. It estimates cashflow, salaries tax, MPF contributions, debt payoff, emergency cash, a life-cover rule of thumb, retirement and savings goals, and it can print a report. Financial figures stay on the device. The account email, latest sign-in time and feature-usage events are stored in Firestore in Hong Kong (asia-east2) to compile usage statistics and to let people sign in. Firebase Authentication is a global Google service, so sign-in data may still be processed outside Hong Kong. The tool is educational only. It is not financial, insurance or MPF advice, and this repository does not add payments or a subscription.

## What it does

An adviser (or the client sitting with them) signs in, enters income, spending, tax details, debts, assets, policies and goals, and sees a live summary in HKD or a manually converted USD view. Sample figures are built in so the page is usable immediately. Nothing typed into the calculator is saved: refresh, close or sign out drops it.

The page is bilingual (繁體中文 and English). Every user-facing string has both versions.

## Features

- Email/password and Google sign-in (Firebase Authentication), with email verification for password accounts
- Cashflow, 50/30/20 check, subscriptions, vehicle costs and an emergency-fund comparison
- Salaries tax estimate for year of assessment 2026/27, and an MPF contribution estimate
- Debt snowball and avalanche illustration, assets and net worth
- Life cover compared with a general rule of thumb (about 10× annual income, plus about 0.5× per dependant) — a comparison, not a recommended sum assured
- Retirement illustration (3% real return assumption) and any number of goals, checked in target-year order
- Charts (ECharts), a what-if slider, and a print/PDF report via the browser print dialog
- Admin statistics for the operator account only (five clicks on the logo): sign-ins and counts of calculator opens, reports and PDF exports
- Personal Information Collection Statement, linked from the sign-in screen and the footer

## Tech

- One static `index.html` (plain JavaScript, no build step)
- [ECharts](https://echarts.apache.org/) 5.5.1 from a CDN
- Firebase JS SDK 11 (Auth + Cloud Firestore compat)
- Firebase Hosting project `nikifintech-wealth`, live at https://nikifintech-wealth.web.app
- `noindex` is set on purpose and is left unchanged

## Data and privacy

Two different kinds of data:

| Data | Where it lives |
| --- | --- |
| Income, expenses, debts, assets, policies, goals, client name, adviser name, and every other calculator figure | Browser memory only. Not written to Firestore, `localStorage`, `sessionStorage`, IndexedDB or a cookie. Cleared on refresh, close or sign-out (the page reloads). |
| Account: email, sign-in provider, verification state, optional display name, account timestamps. Password, if used, is held by Firebase Auth and is not read by this page. | Firebase Authentication (Google), a global service. Sign-in data may be processed outside Hong Kong. |
| `loginLogs/{uid}`: `email`, `lastLogin`. One document per account; each sign-in overwrites it. Written for any signed-in user, including before email verification. | Cloud Firestore in Hong Kong (asia-east2). The signed-in user can create or update only their own document. They cannot read it. |
| `usage_logs`: `uid`, `email`, `action`, `timestamp`. Actions are `open_finance_calculator`, `generate_report` and `export_pdf`. Written only after the account is treated as verified. | Cloud Firestore in Hong Kong (asia-east2). The user can create only their own event. They cannot read, update or delete events. There is no automatic expiry. |

Those Firestore records are kept so the operator can compile usage statistics: how many people sign in, how many times the calculator is opened, and how many reports and PDFs are generated. The account email is also required for sign-in. The page does not use the data for marketing, does not sell it, and does not send it to any third party other than Google, which provides Authentication and Firestore. Firebase Analytics is not loaded.

Only the operator account `yukyiuwan@gmail.com`, and only when Firebase has marked that email verified, can read the two log collections (the admin panel). Other users cannot read anyone's logs, including their own. Access, correction and deletion requests go to yukyiuwan@gmail.com. The in-app notice is the Personal Information Collection Statement.

Firestore security rules in `firestore.rules` match those reads and writes. The admin check uses the verified email on the ID token because this site has no backend that could set a custom claim. A custom claim (`request.auth.token.admin == true`) would be stronger before the tool is offered to other advisers: it cannot be self-assigned, and it keeps working if the operator changes their sign-in email. It needs the Admin SDK, which is intentionally not part of this static site.

The Firebase web API key is in `index.html` because the browser SDK needs it. It is not a secret. Access control is the Auth authorized domains plus `firestore.rules`.

## Run locally

No install is required. From this folder:

```bash
python3 -m http.server 8765
```

Open http://127.0.0.1:8765/ . Sign-in uses the live Firebase project, so a local page still talks to `nikifintech-wealth` unless the hosting domain is not authorized (then Google sign-in fails until `localhost` is added under Authentication → Settings → Authorized domains). Calculator figures are not uploaded either way.

## Deploy

Review the privacy notice and `firestore.rules` before deploying. This repository change does not deploy anything.

```bash
npm install -g firebase-tools
firebase login
firebase deploy --only hosting,firestore:rules --project nikifintech-wealth
```

`firebase deploy --only hosting` updates the website and does **not** publish the rules. The admin panel will keep returning permission errors until the rules are published.

`firestore.rules` and this README are listed in the Hosting ignore list so they are not published as public files.

## Not in this version

Payments, subscriptions and paywalls are intentionally absent. There is no in-app button to delete an account; deletion is by email request, then done in the Firebase console. `usage_logs` are kept until that happens.
