# KeyVault — Password Generation & Security Analysis

A clean, client-side password tool with a realistic security analysis engine. No data is ever sent to a server — everything runs in your browser.

---

## Features

### Generator
- Adjustable length from 6 to 64 characters
- Toggleable character sets: uppercase, lowercase, digits, symbols
- Option to exclude ambiguous characters (0, O, l, 1, I)
- Pronounceable mode for easier-to-remember passwords
- Cryptographically secure generation via the Web Crypto API

### Analyser
- Real-time analysis as you type
- Strength bar with six levels: Critical, Weak, Fair, Good, Strong, Excellent
- Estimated crack time across three realistic attack scenarios
- Actionable recommendations to improve your password

### UI
- Dark and light mode with a toggle
- Clean, minimal design — no clutter, no ads, no tracking
- Fully offline — works without an internet connection once loaded

---

## How the Analysis Works

KeyVault uses a custom engine inspired by zxcvbn (Dropbox's open-source password estimator). Rather than naively computing brute-force entropy from charset size × length, it models how real attackers actually operate.

### Attack model

The engine evaluates every password through multiple attack vectors and uses the cheapest one to compute the final score.

**1. Banned list**
Around 400 of the most common passwords and their leet-speak variants are checked first. "password", "bonjour", "p@ssw0rd" and similar patterns are caught instantly.

**2. Keyboard walk detection**
QWERTY and AZERTY row sequences of 4 or more characters (e.g. "qwerty", "azert", "asdfgh") are flagged as keyboard walks, which attackers enumerate trivially.

**3. Complexity classification**
The engine checks whether the password is genuinely complex or pattern-based:

- **Genuinely complex**: 3+ character categories, at least 10 characters, no word-like letter runs longer than 5 characters, no leet substitution patterns → scored as pure brute-force.
- **Pattern-based**: goes through token-by-token analysis (see below).

**4. Token analysis**
The password is split into tokens: runs of letters, runs of digits, and individual symbols. Each token is scored with the cheapest applicable attack:

- *Word-like alpha tokens* (all-lowercase, TitleCase, CamelCase, ALL CAPS): modelled as a word-list attack using a ~30,000-word vocabulary. Even made-up words are penalised because attackers use language models and word-combination tools like Hashcat rules. Capped at 5×10¹³ guesses regardless of length.
- *Random mixed-case alpha tokens* (e.g. "yTHMnf"): scored as brute-force over 52 characters.
- *Digits*: scored by length (10^n / 2), with a special penalty for year patterns (1900–2099).
- *Symbols*: 33 possible values each.

Token costs are multiplied together for the final guess count.

**5. Leet detection**
Substitutions like 3→e, 0→o, @→a, $→s are reversed before analysis. A password like "Tr0ub4dor" is recognised as the word "Troubador" with leet substitutions applied — not as a random digit sequence.

### Scoring scale

The final guess count is mapped to a 0–100 score on a logarithmic scale:

| Guesses         | Score | Level     |
|-----------------|-------|-----------|
| < 10⁴           | < 8   | Critical  |
| 10⁴ – 10⁸      | 8–25  | Weak      |
| 10⁸ – 10¹³     | 25–46 | Fair      |
| 10¹³ – 10¹⁷    | 46–63 | Good      |
| 10¹⁷ – 10²²    | 63–80 | Strong    |
| > 10²²          | > 80  | Excellent |

### Attack scenarios

Crack times are shown for three scenarios reflecting realistic 2024 hardware:

| Scenario             | Speed     | Context                              |
|----------------------|-----------|--------------------------------------|
| Online (throttled)   | 10/s      | Login form with rate limiting        |
| Offline / bcrypt     | 10M/s     | Leaked bcrypt hash, GPU cluster      |
| Offline / MD5+GPU    | 100B/s    | Leaked MD5 or SHA-1 hash, datacenter |

---

## Calibration examples

| Password                          | Level     | Reason                              |
|-----------------------------------|-----------|-------------------------------------|
| password                          | Critical  | Banned list                         |
| bonjour                           | Critical  | Banned list                         |
| abc123                            | Critical  | Banned list                         |
| Jean2024                          | Weak      | Name + year pattern                 |
| Tr0ub4dor&3                       | Weak      | Leet word detected                  |
| SalutationsJeune                  | Fair      | CamelCase word combination          |
| Boldecacahuetes                   | Fair      | Long lowercase word string          |
| Correct#Horse7                    | Fair      | Known words + digit                 |
| Motdepasse123!                    | Fair      | French word + common suffix         |
| yTHMnf\<DA59:rB                   | Excellent | Genuinely complex, 4 categories     |
| X7#k!2@qP9$mNv3                   | Excellent | Genuinely complex, 4 categories     |
| kH9#mXvP2$nQrL7                   | Excellent | Genuinely complex, 4 categories     |

---

## Usage

KeyVault is a single self-contained HTML file. No build step, no dependencies, no installation required.

1. Download `keyvault.html`
2. Open it in any modern browser
3. Use it fully offline

---

## Technical notes

- Password generation uses `crypto.getRandomValues()` — the same CSPRNG used by password managers
- All analysis is done in-memory in JavaScript; no password ever leaves your device
- The engine runs identically on generated and user-supplied passwords — there is no special bypass for generated passwords
- Compatible with all modern browsers (Chrome, Firefox, Safari, Edge)

---

## License

MIT — do whatever you want with it.
