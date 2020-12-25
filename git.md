# Git

### Посмотреть полный список локальных и удаленных веток
git branch -a

### Насильно протолкнуть изменение, если в удаленном репозитории уже есть новые коммиты поверх текущего
git push -f origin master


### показать все новые файлы 
git ls-files --others --exclude-standard

### удалить все новые файлы
git clean -df

### добавить в add удаленные файлы
git add -u

### отменить последний коммит 
git reset --hard HEAD^

### удалить файл из всех коммитов
git filter-branch --tree-filter 'rm -f passwords.txt' HEAD


### git stash
git stash # добавить текущие незакоммиченные изменения в стек изменений и сбросить текущую рабочую копию до HEAD’а репозитория

git stash list # показать все изменения в стеке

git stash show # показать последнее измененеие в стеке (патч)

git stash apply # применить последнее изменение из стека к текущей рабочей копии

git stash drop # удалить последнее изменение в стеке

git stash pop # применить последнее изменение из стека к текущей рабочей копии и удалить его из стека

git stash clear # очистить стек изменений


### Update all submodules
``
git submodule init
git submodule update
git submodule foreach 'git fetch origin; git checkout $(git rev-parse --abbrev-ref HEAD); git reset --hard origin/$(git rev-parse --abbrev-ref HEAD); git submodule update --recursive; git clean -dfx'
``

### Remove submodule

- Delete the relevant section from the .gitmodules file.
- Stage the .gitmodules changes git add .gitmodules
- Delete the relevant section from .git/config.
- Run git rm --cached path_to_submodule (no trailing slash).
- Run rm -rf .git/modules/path_to_submodule (no trailing slash).
- Commit git commit -m "Removed submodule <name>"
- Delete the now untracked submodule files rm -rf path_to_submodule
