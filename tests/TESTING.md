# Testing Pagination Implementation

This guide explains how to test the pagination implementation in `auto-unapprove.js`.

## 🧪 Test Options

### 1. **Logic Tests** (No API calls)

```bash
# Test pagination logic with simulated data
node test-pagination.js

# Test with mock API responses
node test-mock-pagination.js
```

### 2. **Real Data Tests** (With GitHub API)

```bash
# Set up environment variables
export GITHUB_TOKEN='your_github_token'
export GITHUB_REPOSITORY='owner/repo'
export PR_NUMBER='123'

# Run the test script (DRY_RUN=true is set automatically)
./test-real-pagination.sh

# Or run directly — @actions/core reads inputs from INPUT_* env vars
env \
  "INPUT_GITHUB-TOKEN=$GITHUB_TOKEN" \
  "INPUT_PR-NUMBER=$PR_NUMBER" \
  "INPUT_DRY-RUN=true" \
  GITHUB_REPOSITORY="$GITHUB_REPOSITORY" \
  node ../auto-unapprove.js
```

## 📊 What to Look For

### ✅ Successful Pagination Indicators

1. **Multiple Page Logs**:

   ```text
   📄 Fetching page 1...
   📄 Page 1: 100 files
   📄 Fetching page 2...
   📄 Page 2: 45 files
   ```

2. **Correct Total Counts**:

   ```text
   📁 All changed files in PR (145):
   ```

3. **No Missing Files**:
   - All files should be listed
   - No "only 30 files" limitations

### ❌ Pagination Issues to Watch For

1. **Single Page Only** (for large PRs):

   ```text
   📄 Fetching page 1...
   📄 Page 1: 30 files  # Should be 100 if more files exist
   ```

2. **Missing Files**:

   - GitHub shows 150 files but script only finds 30
   - Incomplete file analysis

3. **API Errors**:
   ```text
   ❌ Failed to fetch PR files page 2: 404
   ```

## 🔍 Finding Test PRs

### Good Test Candidates

- **Small PRs** (<30 files): Should show 1 page
- **Medium PRs** (30-100 files): Should show 1 page with 100 items
- **Large PRs** (>100 files): Should show multiple pages

### How to Find Large PRs

1. Go to your repository on GitHub
2. Look at the "Files changed" count in PR list
3. Target PRs with 100+ changed files
4. Check PR description for file count

## 🚀 Testing Scenarios

### Scenario 1: Small PR (<30 files)

```bash
env "INPUT_GITHUB-TOKEN=$GITHUB_TOKEN" "INPUT_PR-NUMBER=small-pr-number" "INPUT_DRY-RUN=true" GITHUB_REPOSITORY="$GITHUB_REPOSITORY" node ../auto-unapprove.js
```

**Expected**: Single page, all files found

### Scenario 2: Medium PR (30-100 files)

```bash
env "INPUT_GITHUB-TOKEN=$GITHUB_TOKEN" "INPUT_PR-NUMBER=medium-pr-number" "INPUT_DRY-RUN=true" GITHUB_REPOSITORY="$GITHUB_REPOSITORY" node ../auto-unapprove.js
```

**Expected**: Single page with 100 items, all files found

### Scenario 3: Large PR (>100 files)

```bash
env "INPUT_GITHUB-TOKEN=$GITHUB_TOKEN" "INPUT_PR-NUMBER=large-pr-number" "INPUT_DRY-RUN=true" GITHUB_REPOSITORY="$GITHUB_REPOSITORY" node ../auto-unapprove.js
```

**Expected**: Multiple pages, correct total count

## 🛡️ Safety Testing

### Always Start with Dry Run

Pass `INPUT_DRY-RUN=true` (the default) before running directly, or use `test-real-pagination.sh` which sets it automatically.

### Check Output Before Live Run

1. Verify all files are found
2. Confirm pagination worked
3. Review dismissal targets
4. Only then set `INPUT_DRY-RUN=false`

## 📈 Performance Testing

### Monitor API Call Count

- Small PR: ~3-4 API calls
- Large PR: ~6-10 API calls (depending on pages)

### Check Response Times

- Each page should take 1-3 seconds
- Total time should be reasonable for file count

## 🔧 Troubleshooting

### Common Issues

1. **"Only 30 files found"**:

   - Pagination not working
   - Check for API errors
   - Verify page size parameter

2. **"Failed to fetch" errors**:

   - Check GitHub token permissions
   - Verify repository access
   - Check PR number validity

3. **Incomplete results**:
   - Look for pagination logs
   - Check if all pages were fetched
   - Verify break conditions

### Debug Commands

```bash
# Check environment variables
echo "Token: ${GITHUB_TOKEN:0:10}..."
echo "Repo: $GITHUB_REPOSITORY"
echo "PR: $PR_NUMBER"

# Test API access
curl -H "Authorization: Bearer $GITHUB_TOKEN" \
     "https://api.github.com/repos/$GITHUB_REPOSITORY/pulls/$PR_NUMBER/files?per_page=1"
```

## ✅ Success Criteria

Your pagination implementation is working correctly if:

1. ✅ **All files are found** (matches GitHub PR file count)
2. ✅ **Multiple pages are fetched** (for large PRs)
3. ✅ **No API errors** occur during pagination
4. ✅ **Performance is reasonable** (not too slow)
5. ✅ **Results are consistent** across different PR sizes

## 🎯 Next Steps

After confirming pagination works:

1. Test with real dismissals (`INPUT_DRY-RUN=false`)
2. Monitor GitHub Actions logs
3. Verify review dismissals are correct
4. Check that all code owners are properly identified

---

**💡 Tip**: The mock tests (`test-mock-pagination.js`) are perfect for quick validation without needing real GitHub data or tokens.
