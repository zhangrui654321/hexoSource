##### Git global setup

```
git config --global user.name "Administrator"
git config --global user.email "admin@example.com"
```

##### Create a new repository

```
git clone ssh://git@192.168.2.2:222/root/newbeemallplus.git
cd newbeemallplus
git switch -c main
touch README.md
git add README.md
git commit -m "add README"
git push -u origin main
```

##### Push an existing folder

```
cd existing_folder
git init --initial-branch=main
git remote add origin ssh://git@192.168.2.2:222/root/newbeemallplus.git
git add .
git commit -m "Initial commit"
git push -u origin main
```

##### Push an existing Git repository

```
cd existing_repo
git remote rename origin old-origin
git remote add origin ssh://git@192.168.2.2:222/root/newbeemallplus.git
git push -u origin --all
git push -u origin --tags
```