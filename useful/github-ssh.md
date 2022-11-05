генерим ключики:
```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

пароль можно и поменять: 
```
ssh-keygen -p
```

надо добавить ключи в ssh-агент:
```
eval $(ssh-agent -s)
Agent pid 59566
```

```
ssh-add ~/.ssh/id_rsa
```  

копируем `id_rsa.pub` в гитхаб

активируем:  
```
ssh -T git@github.com
```  

далее нам нужно настроить `~/.zshrc`, он будет содержать инициализационные настройки для консоли и запускать ssh-агент, который будет включать ваши ключи, запоминать сессию:  

```
# Note: ~/.ssh/environment should not be used, as it
#       already has a different purpose in SSH.

env=~/.ssh/agent.env

# Note: Don't bother checking SSH_AGENT_PID. It's not used
#       by SSH itself, and it might even be incorrect
#       (for example, when using agent-forwarding over SSH).

agent_is_running() {
    if [ "$SSH_AUTH_SOCK" ]; then
        # ssh-add returns:
        #   0 = agent running, has keys
        #   1 = agent running, no keys
        #   2 = agent not running
        ssh-add -l >/dev/null 2>&1 || [ $? -eq 1 ]
    else
        false
    fi
}

agent_has_keys() {
    ssh-add -l >/dev/null 2>&1
}

agent_load_env() {
    . "$env" >/dev/null
}

agent_start() {
    (umask 077; ssh-agent >"$env")
    . "$env" >/dev/null
}

if ! agent_is_running; then
    agent_load_env
fi

# if your keys are not stored in ~/.ssh/id_rsa or ~/.ssh/id_dsa, you'll need
# to paste the proper path after ssh-add
if ! agent_is_running; then
    agent_start
    ssh-add
elif ! agent_has_keys; then
    ssh-add
fi

unset env
```

надо сменить адреса реп, проверяются командой `git remote -v`. Для смены адресов git есть специальная команда: `set-url`, ввести следующее:  

```
$ git remote set-url origin git@github.com:xotohop/knowledge-base.git
```