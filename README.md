# Activity Importer: Recreate Commit History with Shortstat Data

This repository recreates my commit history from inactive accounts to preserve my contribution record. This includes commit messages, timestamps, and shortstat data (number of files changed, insertions, deletions), without transferring any actual code from the original repository. This process is ideal for situations where the commit history needs to be preserved or documented without transferring proprietary or sensitive code between repositories.

## Methodology

### 1. Extract Commit History from the Old Repository

To begin, extract the commit history from the original repository using the following command:

```bash
cd path/to/old-repo

# Extract commit hash, message, date, and shortstat data
git log --author="your-email@example.com" --pretty=format:"%H|%s|%ad" --date=iso --shortstat > commits_shortstat.txt
```

This command generates a `commits_shortstat.txt` file containing each commit's hash, message, date, and associated shortstat data (files changed, insertions, and deletions).

### 2. Format the Shortstat Data

Next, use a Python script to parse and format the commit data from `commits_shortstat.txt` into a more usable format, which includes the shortstat information:

```python
import re

with open('commits_shortstat.txt', 'r') as infile, open('commits_to_replay.txt', 'w') as outfile:
    current_commit = ""
    for line in infile:
        if '|' in line:
            if current_commit:
                outfile.write(current_commit + "\n")
            current_commit = line.strip()
        elif "file changed" in line or "files changed" in line:
            # Extract the shortstat data
            shortstat = re.sub(r'\s+', ' ', line.strip())
            current_commit += f" | {shortstat}"
    if current_commit:
        outfile.write(current_commit + "\n")
```

This script reads through `commits_shortstat.txt`, extracts relevant data, and writes it to `commits_to_replay.txt`.

### 3. Replay Commits in the New Repository

With the formatted commit data ready, proceed to recreate the commits in the new repository, appending the shortstat data to each commit message:

```bash
cd path/to/new-repo

# Initialize a new file to track replayed commits
touch commits-yourproject.txt
git add commits-yourproject.txt
git commit -m "Initialize commits-yourproject.txt for recording replayed commits"

# Replay each commit in chronological order
while IFS='|' read -r commit_hash commit_message commit_date shortstat; do
    # Prepend commit details including shortstat to the commits-yourproject.txt file
    echo "$commit_hash | $commit_message | $commit_date | $shortstat" | cat - commits-yourproject.txt > temp_file && mv temp_file commits-yourproject.txt

    # Stage the changes including the updated commits-yourproject.txt
    git add commits-yourproject.txt

    # Commit with the original message and date
    GIT_COMMITTER_DATE="$commit_date" git commit -m "$commit_message - $shortstat" --date "$commit_date"

done < <(sort -t'|' -k3 commits_to_replay.txt)

# Push to the new repository
git push origin main
```

### 4. Push to the New Repository

Finally, push the recreated commit history to the new repository:

```bash
git push origin main
```

## Result

The commit history, including shortstat data, is now recreated in the new repository. All commits are documented in the `commits-yourproject.txt` file, with the shortstat data appended to each commit message for documentation purposes.

## Notes

- **Preservation of Commit History:** The original commit dates, messages, and shortstat data are preserved without transferring the actual code.
- **Chronological Order:** Commits are replayed in chronological order to maintain accurate history.
- **Documentation:** The `commits-*.txt` files in the repository serve as a log of all replayed commits.
