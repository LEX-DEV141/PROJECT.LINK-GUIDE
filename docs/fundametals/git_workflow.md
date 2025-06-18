# 🌿 Git Workflow - LEX DEV

## 🎯 Tujuan Pembelajaran
- Memahami konsep version control dan Git
- Menguasai Git workflow yang digunakan di LEX DEV
- Mampu berkolaborasi efektif menggunakan Git
- Memahami best practices dalam commit dan branching

## 📚 Konsep Dasar Git

### Apa itu Git?
Git adalah sistem version control terdistribusi yang memungkinkan tim untuk:
- Melacak perubahan kode dari waktu ke waktu
- Berkolaborasi tanpa konflik
- Membuat backup otomatis dari setiap perubahan
- Rollback ke versi sebelumnya jika diperlukan

### Konsep Penting
- **Repository (Repo)**: Folder project yang di-track oleh Git
- **Commit**: Snapshot dari perubahan kode pada waktu tertentu
- **Branch**: Cabang development yang terpisah dari main branch
- **Merge**: Menggabungkan perubahan dari satu branch ke branch lain
- **Pull Request (PR)**: Permintaan untuk merge perubahan ke branch utama

## 🔧 Git Workflow LEX DEV

### Branch Strategy
Kami menggunakan **Git Flow** yang dimodifikasi:

```
main/master     ←→  Production-ready code
├── develop     ←→  Integration branch
    ├── feature/xxx  ←→  New features
    ├── bugfix/xxx   ←→  Bug fixes
    └── hotfix/xxx   ←→  Critical fixes
```

### Naming Convention
- `feature/LEX-123-user-authentication`
- `bugfix/LEX-456-fix-login-error`
- `hotfix/LEX-789-critical-security-patch`

Format: `type/ticket-number-short-description`

## 🛠️ Implementasi Praktis

### 1. Setup Awal
```bash
# Clone repository
git clone https://github.com/lexdev/project-name.git
cd project-name

# Setup user identity
git config user.name "Nama Anda"
git config user.email "email@lexdev.com"

# Switch ke develop branch
git checkout develop
git pull origin develop
```

### 2. Membuat Feature Branch
```bash
# Buat branch baru dari develop
git checkout develop
git pull origin develop
git checkout -b feature/LEX-123-user-login

# Mulai coding...
```

### 3. Commit Guidelines
```bash
# Add files
git add .

# Commit dengan pesan yang jelas
git commit -m "feat: add user login functionality

- Implement login form validation
- Add JWT token handling
- Create login API endpoint

Resolves: LEX-123"
```

### 4. Push dan Pull Request
```bash
# Push branch ke remote
git push origin feature/LEX-123-user-login

# Buat Pull Request melalui GitHub/GitLab
# Target: develop branch
```

## ✅ Best Practices

### Commit Message Format
```
type(scope): short description

Longer description if needed

- Detail 1
- Detail 2

Resolves: TICKET-NUMBER
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Code style changes
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks

### Do's ✅
- Commit frequently dengan perubahan kecil
- Write descriptive commit messages
- Pull latest changes sebelum push
- Test code sebelum commit
- Use meaningful branch names
- Delete branch setelah merge

### Don'ts ❌
- Jangan commit ke main/master directly
- Jangan commit code yang broken
- Jangan push credentials atau sensitive data
- Jangan force push ke shared branches
- Jangan commit generated files (node_modules, build/, etc.)

## 🧪 Latihan Praktis

### Exercise 1: Basic Workflow
1. Clone repository contoh
2. Buat feature branch untuk "add contact form"
3. Buat 3 commits dengan perubahan bertahap
4. Push dan buat pull request

### Exercise 2: Conflict Resolution
1. Simulasi merge conflict
2. Resolve conflict manually
3. Complete merge process

### Exercise 3: Git History
1. Explore git log dengan berbagai format
2. Use git blame untuk trace code changes
3. Practice git reset dan git revert

## 🔧 Tools & Commands

### Useful Git Commands
```bash
# Status dan info
git status                  # Check working directory status
git log --oneline          # Compact commit history
git diff                   # Show unstaged changes
git diff --cached          # Show staged changes

# Branch management
git branch -a              # List all branches
git branch -d branch-name  # Delete local branch
git remote prune origin    # Clean up remote branches

# Undo changes
git checkout -- file.txt  # Discard unstaged changes
git reset HEAD file.txt    # Unstage file
git revert commit-hash     # Revert specific commit
```

### Git Aliases (Optional)
```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'
```

## 📖 Resources Tambahan
- [Git Documentation](https://git-scm.com/doc)
- [Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials)
- [GitHub Flow Guide](https://guides.github.com/introduction/flow/)
- [Conventional Commits](https://www.conventionalcommits.org/)

## ❓ FAQ

**Q: Kapan harus membuat branch baru?**
A: Setiap kali mulai feature baru, bug fix, atau task yang terpisah.

**Q: Boleh push langsung ke main branch?**
A: Tidak. Selalu gunakan pull request process.

**Q: Bagaimana handling merge conflicts?**
A: Communicate dengan tim, resolve secara manual, test thoroughly sebelum merge.

**Q: Berapa lama branch boleh hidup?**
A: Idealnya tidak lebih dari 1-2 minggu. Feature yang besar dipecah menjadi smaller chunks.

## 🎯 Checklist Keberhasilan
- [ ] Dapat clone dan setup repository
- [ ] Dapat membuat dan manage branches
- [ ] Dapat write good commit messages
- [ ] Dapat create dan review pull requests
- [ ] Dapat resolve basic merge conflicts
- [ ] Memahami Git workflow tim LEX DEV
