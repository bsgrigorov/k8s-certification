# Exam day

## Shell setup commands
Run these at the beginning of your exam session
```
alias k=kubectl
alias ka='k apply -f $@' # 42
alias kd='k describe $@'
printf 'set sw=2\nset ts=2\nset expandtab' > ~/.vimrc
export dr="--dry-run=client -o yaml"
```

## Vim help
Vim commands to speed up your YAML editing/
```
>2> - indent
<3< - remove indent
d3d - delete 
dd - delete
y4y - copy
p - paste after
P - paste before
u - undo
i - insert
A - insert mode end of line
^ - start of line
$ - end of line
I - insert mode in start of line
/term - search for term
o - insert line below and enter insert mode in it
O - insert line above and enter insert mode in it
```