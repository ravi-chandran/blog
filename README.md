# RC Notebook

## Installation
- Create GitHub repository called `blog`.

- Then clone the `blog` repository and install Nikola in its root folder.
```bash
cd ~/repos/blog
git clone git@github.com:ravi-chandran/blog.git
cd blog
python3 -m venv nikola
source ./nikola/bin/activate
python3 -m pip install --upgrade pip setuptools wheel
python3 -m pip install --upgrade "Nikola[extras]"
```

- Add these to `.bash_aliases` for convenience: 
```bash
alias blog="cd ~/repos/blog"
alias nik="source ./nikola/bin/activate"
```

- If it's the first time, initialize the repository as a Nikola site:
```bash
cd ~/repos/blog
source ./nikola/bin/activate
nikola init .
```

## Usage

### Create New Post
```bash
blog  # gets to my repo
nik   # activates the nikola virtual environment

# create a folder for the category if it doesn't already exist
mkdir -p posts/somecategory

# create post in the appropriate folder under posts - each folder represents a category
nikola new_post -t "This is my title" posts/somecategory
```

### Build and Review Post
```bash
nikola build
nikola auto -b  # or use: nikola serve -b
```

### Publish
1. Commit the post's files to `git`

2. Deploy the post to the blog:
```bash
nikola github_deploy
```