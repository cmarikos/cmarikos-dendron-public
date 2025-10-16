Going to work with [Open Elections](https://github.com/openelections/openelections-core) for the [[2024-c4-campaign-ids]] file join Rich needs. Currently I don't have election results in a format I can use my [[projects.az-precinct-name-conversion-script]] on. Without a unified precinct code files can't be joined for Rich in QGIS.

##### Install pipx snf pipenv

Run in the terminal
 %%Aside: I'd like to email Max Howell a kiss on the mouth for every time homebrew made my life easier%%
```zsh
	cmarikos@MacBookAir ~ % brew install pipx
```

Then run
```zsh
cmarikos@MacBookAir ~ % pipx install pipenv
```

Then
```zsh
cmarikos@MacBookAir ~ % pipx ensure path
```

And
```zsh
cmarikos@MacBookAir ~ % source ~/.zshrc
```

And finally
```zsh
	cmarikos@MacBookAir ~ % pipenv --version
**pipenv**, version 2024.4.1
```

Yay!

##### Make a virtual environment then fork the Open Elections repo

Run
```zsh
cmarikos@MacBookAir ~ % pipenv install --dev
```

Then
```zsh 
cmarikos@MacBookAir ~ % pipenv shell
```


##### Fork the repo
Gonna set up osxkeychain first
```zsh
(cmarikos) cmarikos@MacBookAir ~ % git config --global credential.helper osxkeychain
```

Verify that is worked
```zsh
(cmarikos) cmarikos@MacBookAir ~ % git config --global credential.helper

osxkeychain
```
Yay!


```zsh
(cmarikos) cmarikos@MacBookAir ~ % git clone https://github.com/cmarikos/openelections-core.git

```

Generate a PAT then enter
```zsh
(cmarikos) cmarikos@MacBookAir ~ % git clone https://github.com/cmarikos/openelections-core.git

Cloning into 'openelections-core'...

Username for 'https://github.com': cmarikos

Password for 'https://cmarikos@github.com':
```

Something went wrong
```zsh
remote: Repository not found.

fatal: repository 'https://github.com/cmarikos/openelections-core.git/' not found
```

So this worked? Maybe
```zsh
(cmarikos) cmarikos@MacBookAir ~ % git clone https://github.com/openelections/openelections-core
```

I'm stupid, like genuinely dumb. I didn't click 'Fork Repo' so obviously the 'https://github.com/cmarikos/openelections-core.git/' didn't work. All good now. I'm learning. Maybe I shouldn't be forking repos if I don't know how to fork repos, but gotta start somewhere  I guess.