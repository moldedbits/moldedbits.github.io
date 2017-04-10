---
layout: post
title:  "Accessing Git APIs in Ruby"
date:   2015-09-15 11:11:11
author: amit
categories: Technical Ruby
comments: true
---
Without a doubt we all know that Git is one of the most famous version control system. Here at moldedbits we use [GitHub][5c4dd6b1] for each of our project. Some use git directly from command line, while other prefer GUI tools like [SourceTree][c30c4e2e].

### We all know this. Tell me something new.

Yeah, you all know this. But what if you have to provide API for your own git repo or as in our case we wanted to create a document versioning system using the power of git. When I was trying to solve this problem there were multiple options, but [libgit2][5f46323f] stands out of all. As a pure C implementation it provide complete power of core modules of Git. Previously [Grit][bf9bf480] was popular for providing the Ruby bindings for libgit2. But Grit is no longer supported and currently Rugged gem provides the updated bindings to libgit2 in Ruby.

### Rugged Gem
As described on its [github repository][3ecfc539], Rugged is a wrapper upon [libgit2][5f46323f]. This provide all the functionality of libgit2 with the beauty of Ruby language.

### libgit2
Let's first clear out what is [libgit2][7199ea7a]. libgit2 is pure C implementation of core git method, to provide as linkable library with git API. libgit2 allow us to write Git application using its native API. libgit2 bindings to multiple languages are available and can be found at [libgit2-bindings][09f9859e].

## Starting out with Rugged

### Let's Install

Rugged can be installed directly using gem install.

`$ gem install rugged`

As rugged is build upon libgit2, so you need to have `cmake` and `pkg-config` to be able to build `libgit2`. Using `homebrew` you can install `cmake`.

`$ brew install cmake`

To load Rugged you need to require it using

`require rugged`

### Show me how to use

Rugged provide us access to the Git repository using multiple Git API. You can create repo, read repo, create commits, walk tree, create tags, check diff and many more. I will be covering some of the basic functionality which can help you to understand the rugged.

#### Create Git repository
```Ruby
repo = Rugged::Repository.init_at('path/to/repo', :bare)
 #<Rugged::Repository:70262130283440 {path: "/path/to/repo.git/"}>
```

Using `Repository` class `init_at` method you can instantiate a git repository. The first parameter is the path where you want to create repo. The second argument `:bare` is boolean, if you want to create new repo a [bare][712a3aeb] repo or not. Above will return the object of `Repository` class.

In case you already have a repo and you access it using `new` method.

```Ruby
repo = Rugged::Repository.new('../abc')
 #<Rugged::Repository:70262121865280 {path: "/path/to/repo.git/"}>
```

#### Repository Object

Once you have Repository object you can get multiple information or access different methods.

```Ruby
# Repository state value
repo.bare?
# => false
repo.empty?
# => true
repo.head_detached?
# => false

# Empty repository will give error on the following method call, as reference to the head not exist in empty repository.
# Repository Head reference
ref_head = repo.head

# From the returned ref, you can also access the `name`, `target`, and target SHA:
ref_head.name
# => "refs/heads/master"
ref_head.target
# => #<Rugged::Commit:2228467250 {message: "helpful message", tree: #<Rugged::Tree:2228467260 {oid: 5d6f29220a0783b8085134df14ec4d960b6c3bf2}>}>
ref_head.target_id
# => "2bc6a70483369f33f641ca44873497f13a15cde5"
```

#### Commit to Repository

To create a commit you first need to write to the index of the repository. First you will create a file which you want to save to repo and create a commit.

```Ruby
# Open or create a new file
file = open('/path/to/repo/test.md', 'w')
# Write some content to the file
file << "This is new test content for commit."
# Close the file
file.close

# Fetch the current Index of the repository
index = repo.index
# Add the file to current Index
index.add(filename)
# Writes the index object from memory back to the disk, persisting all changes.
index.write
```

The above code will add the newly created file or the changed content of the file to the repository's database and write all the changes to the disk. You can also write direct to the repository's database and after that write the changes to the disk.

```Ruby
# Write the blob object to the repository's database
oid = repo.write("This is a blob.", :blob)
# Get the current index object of repository
index = repo.index
# Clears the current index and starts the index on top of tree
index.read_tree(repo.head.target.tree)
# Add the changes to the index
index.add(:path => "README.md", :oid => oid, :mode => 0100644)
index.write
```

```Ruby
# Create an option hash to pass the details of commit
options = {}
# Writes the index to the current repository's tree, In case of any conflict this will fail
options[:tree] = index.write_tree(repo)
# Give author details
options[:author] = { :email => "author@mail.com" , :name => "Author name", :time => Time.now }
# Give Committer details
options[:committer] = { :email => "commiter@mail.com", :name => 'Committer name', :time => Time.now }
# Commit Message
options[:message] ||= "First commit using Rugged"
# Define current parent for the commit
options[:parents] = repo.empty? ? [] : [ repo.head.target ].compact
# Which reference needs to be updated after commit
options[:update_ref] = 'HEAD'
```

Each line of code is explained using inline comments. After creating the commit options you need to create a commit using `Commit` class.

```Ruby
# This will return the SHA of the newly created commit.
Rugged::Commit.create(repo, options)

```

#### Commit Object

After creating a commit if you want to examine the commit, you can get the commit object using the SHA of the commit.

```Ruby
commit = repo.lookup('a0ae5566e3c8a3bddffab21022056f0b5e03ef07')
# => #<Rugged::Commit:2245304380>

commit.message
# => "First commit using Rugged`\n"

commit.time
# => Tue Sep 15 21:23:25 -0700 2015

commit.author
# => {:email=>"author@mail.com", :name=>"Author name", :time=>Tue Sep 15 21:23:25 -0700 2015}

commit.tree
# => #<Rugged::Tree:2245269740>

commit.parents
```
You can also use `Walker` class to traverse through set of commits. You first create a new walker object than set a sorting strategy for the walker. You push head SHAs onto the walker, and then call next to get a list of the reachable commit objects one at a time. You can also hide() commits if you are not interested in anything beneath them.

```Ruby
walker = Rugged::Walker.new(repo)
walker.sorting(Rugged::SORT_TOPO | Rugged::SORT_REVERSE) # optional
walker.push(hex_sha_interesting)
walker.hide(hex_sha_uninteresting)
walker.each { |c| puts c.inspect }
walker.reset
```

#### Diff Object

There are multiple ways to get the diff. But simplest one is getting the diff between two commits  or the index/staging and current working directory.

```Ruby
# Diff between two subsequent commits
diff_commits = commit_object.parents[0].diff(commit_object)

# Diff between index/staging and current working directory
diff_index = repository.index.diff

```
After getting the diff Object you can execute multiple operation on this.

```Ruby
# Get patch
diff.patch
=> "diff --git a/foo1 b/foo1\nnew file mode 100644\nindex 0000000..81b68f0\n--- /dev/null\n+++ b/foo1\n@@ -0,0 +1,2 @@\n+abc\n+add line1\ndiff --git a/txt1 b/txt1\ndeleted file mode 100644\nindex 81b68f0..0000000\n--- a/txt1\n+++ /dev/null\n@@ -1,2 +0,0 @@\n-abc\n-add line1\ndiff --git a/txt2 b/txt2\nindex a7bb42f..a357de7 100644\n--- a/txt2\n+++ b/txt2\n@@ -1,2 +1,3 @@\n abc2\n add line2-1\n+add line2-2\n"

# Get delta (faster, if you only need information on what files changed)
diff.each_delta{ |d| puts d.inspect }
#<Rugged::Diff::Delta:70144372137380 {old_file: {:oid=>"0000000000000000000000000000000000000000", :path=>"foo1", :size=>0, :flags=>6, :mode=>0}, new_file: {:oid=>"81b68f040b120c9627518213f7fc317d1ed18e1c", :path=>"foo1", :size=>14, :flags=>6, :mode=>33188}, similarity: 0, status: :added>

# Detect renamed files
# Note that the status field changed from :added/:deleted to :renamed
diff.find_similar!
diff.each_delta{ |d| puts d.inspect }
#<Rugged::Diff::Delta:70144372230920 {old_file: {:oid=>"81b68f040b120c9627518213f7fc317d1ed18e1c", :path=>"txt1", :size=>14, :flags=>6, :mode=>33188}, new_file: {:oid=>"81b68f040b120c9627518213f7fc317d1ed18e1c", :path=>"foo1", :size=>14, :flags=>6, :mode=>33188}, similarity: 100, status: :renamed>

# Merge one diff into another (mutating the first one)
diff1.merge!(diff2)

# Write a patch into a file (or any other object responding to write)
# Note that the patch as in diff.patch will be written, it won't be applied
file = File.open('/some/file', 'w')
diff.write_patch(file)
file.close
```

### Further References

You can get more details about this gem on the Github repo of [Rugged][3ecfc539].
Rubydoc for the same is available at [Rugged Doc][0fad1dd5].

Happy coding!

The moldedbits Team

[0fad1dd5]: http://www.rubydoc.info/github/libgit2/rugged "Rugged Doc"
[bf9bf480]: https://github.com/mojombo/grit/ "Grit"
[712a3aeb]: http://www.saintsjd.com/2011/01/what-is-a-bare-git-repository/ "bare"
[3ecfc539]: https://github.com/libgit2/rugged "Rugged"
[5f46323f]: https://github.com/libgit2/libgit2 "libgit2"
[7199ea7a]: https://github.com/libgit2/libgit2 "libgit2"
[09f9859e]: https://github.com/libgit2/libgit2#language-bindings "libgit2-bindings"
[21fa2960]: https://github.com/mojombo/grit/ "Grit Gem"
[5c4dd6b1]: https://github.com "GitHub"
[c30c4e2e]: https://www.sourcetreeapp.com "Source Tree"

{% if page.comments %}
{% include disqus.html %}
{% endif %}
