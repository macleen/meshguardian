
# 🧩 Git Workflow Cheat Sheet

A reference for choosing the right Git strategy for your team, project, or personal hacking.

---

## 👨‍💻 1. Solo Dev: Quick & Dirty

Use for: personal projects, quick prototyping

```bash
git init
git add .
git commit -m "quick commit"
git push origin main
```

✅ Fast  
❌ Not scalable or collaborative

---

## 👥 2. Feature Branch Workflow

Use for: team projects, open source

```bash
git checkout -b feature/your-feature
# make changes
git add .
git commit -m "feat: your message"
git push origin feature/your-feature
```

Then open a pull request.

✅ Keeps `main` clean  
✅ Supports code review and CI/CD

---

## 🔁 3. GitHub Flow

Ideal for continuously deployed apps

- Create branch
- Work + test
- Open PR to `main`
- Auto-deploy

Branches:
- `fix/issue-id`
- `feat/module-name`

✅ Lightweight  
✅ Auto-deploys  
❌ Requires solid test coverage

---

## 🚀 4. Gitflow (Advanced Teams)

Use for: versioned releases, staging pipelines

```bash
main        → production
develop     → dev
feature/*   → features
release/*   → prepping a release
hotfix/*    → emergency fixes
```

✅ Powerful for complex pipelines  
❌ Overkill for small teams

---

## ✨ 5. Rebase & Squash

Use for: perfectly clean history

```bash
git checkout main
git pull
git checkout feature-branch
git rebase main
git push --force
```

To squash commits:

```bash
git rebase -i HEAD~3
```

✅ Clean history  
⚠️ Risky if used incorrectly on shared branches

---

## 🛠️ Git Tips

```bash
git log --oneline --graph --all   # Visualize history
git stash                         # Save changes without committing
git reset HEAD~1                  # Undo last commit
git diff                          # View file changes
git pull --rebase                 # Avoid merge commits
```

---

## 📊 Workflow Comparison

| Workflow         | Easy | Clean | Team-Friendly | Rollback | Use Case                 |
|------------------|------|-------|----------------|----------|--------------------------|
| Quick & Dirty     | ✅   | ❌    | ❌             | ❌       | Solo projects            |
| Feature Branch    | ✅   | ✅    | ✅             | ✅       | OSS / Team workflows     |
| GitHub Flow       | ✅   | ✅    | ✅             | ✅       | CI/CD, web apps          |
| Gitflow           | ❌   | ✅    | ✅             | ✅       | Complex projects         |
| Rebase + Squash   | ❌   | ✅✅   | ⚠️             | ❌       | Polished PRs & OSS       |

---

📘 For more help, visit: [https://meshguardian.com](https://meshguardian.com)
