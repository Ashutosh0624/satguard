# Git Errors

Git, GitHub aur CI se jude saare errors yahan hain, sabse naya sabse upar.

### GIT-001 — Push rejected: PAT mein `workflow` scope nahi tha

- Date: 2026-11-01 · Project: aerosat-health · Severity: Medium · Status: Fix identified

**Challenge (kya hua):** Naye scaffold ko `git push origin develop` se push kiya to GitHub ne poora push reject kar diya:

```text
! [remote rejected] develop -> develop (refusing to allow a Personal Access Token to create or update workflow `.github/workflows/ci.yml` without `workflow` scope)
error: failed to push some refs to 'https://github.com/Ashutosh0624/aerostat-health.git'
```

**Root cause (kyun hua):** GitHub Actions workflows repo ke secrets ke saath code run karte hain. Isliye GitHub kisi bhi token ko workflow file banane ya badalne tabhi deta hai jab us token mein explicitly `workflow` scope ho. Mere Personal Access Token (PAT) mein `repo` scope tha, `workflow` nahi. Scaffold mein `.github/workflows/ci.yml` tha, isliye pura push reject ho gaya.

**Resolution (kya kiya):** Root cause identify karke teen working fixes nikale. Inme se koi ek apply karna hai:

1. **GitHub CLI (sabse aasaan):** gh ke token mein `workflow` scope add karo aur git ko wahi token use karne do.
2. **SSH remote (long-term best):** SSH keys pe token-scope ka restriction lagta hi nahi.
3. **Naya PAT:** Classic token mein `repo` + `workflow`; fine-grained token mein *Contents* aur *Workflows* dono Read & write. Phir purana cached token hatao.

```bash
# Option 1 — GitHub CLI
gh auth refresh -h github.com -s workflow
gh auth setup-git
git push origin develop

# Option 2 — SSH
ssh -T git@github.com
git remote set-url origin git@github.com:Ashutosh0624/aerosat-health.git
git push origin develop

# Option 3 — naya PAT ke baad purana cached token hatao
printf "protocol=https\nhost=github.com\n\n" | git credential reject
git push origin develop   # password ki jagah naya token
```

**Applied fix:** Kaunsa option use kiya, yahan likho, aur push success hone ke baad Status ko Resolved karo.

**Prevention (aage kaise bachein):** Naye machine ya repo setup pe default SSH ya `gh auth login` use karo. PAT banana pade to checklist: `repo` + `workflow`, aur expiry date calendar mein daalo.

**Learning / interview angle:** CI scaffold push karte waqt token-scope rejection mila. Root cause GitHub ka least-privilege security model tha, jisme workflow files ke liye alag scope chahiye. Ise SSH/scoped auth se fix kiya aur team setup checklist mein add kiya.

**Same push mein dikhe related issues:**

- Remote repo ka naam `aerostat-health` hai (typo), project `aerosat-health` hai. Fix: `gh repo rename aerosat-health --repo Ashutosh0624/aerostat-health` + `git remote set-url`.
- `develop` pe push se CI trigger nahi hota, kyunki workflow sirf `main` aur PRs pe chalta hai. Fix: `main` push karo, ya `ci.yml` mein `branches: [main, develop]`.
