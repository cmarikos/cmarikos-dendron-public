
In this document, I will outline my experience installing #parsons. I’m not a software developer, but I’m motivated to learn and improve my technical skills. Most of my expertise lies in data analysis and visualization, particularly in SQL, and I'm improving my Python skills over time. Although I'm not yet fully comfortable with all the technical aspects of Parsons, I’m excited about the possibilities it offers and want to contribute to the project in the future.

I’m following the official installation guide provided by Parsons: [Installation Guide](https://www.parsonsproject.org/pub/installation/release/10).

### Step 1: Check my python version
```zsh
cmarikos@Christinas-MacBook-Air ~ % python3 --version

Python 3.13.1
```

According to the guide parsons needs 3.8 - 3.10. I will install a compatible version (3.10.13) using **pyenv**.

Sophia Alice from the dev group recommended UV for package and dependency management, I may revisit this later
- [UV docs]( https://docs.astral.sh/uv/)

### Step 2: install pyenv for version management

```zsh
cmarikos@Christinas-MacBook-Air ~ % brew install pyenv
```

### Step 3: Install Python 3.10.13

Using **pyenv**, I’ll install Python 3.10.13

```zsh
cmarikos@Christinas-MacBook-Air ~ % pyenv install 3.10.13
```

### Step 4: Create and Activate a Virtual Environment

Next, I’ll create a new virtual environment with Python 3.10.13 to keep the installation isolated

```zsh
cmarikos@Christinas-MacBook-Air ~ % pyenv local 3.10.13
cmarikos@Christinas-MacBook-Air ~ % python3.10 -m venv parsons_env
cmarikos@Christinas-MacBook-Air ~ % source parsons_env/bin/activate

(parsons_env) cmarikos@Christinas-MacBook-Air ~ %
```
The prompt should show the virtual environment is active `(parsons_env)`

### Step 5: Install Parsons

With the virtual environment active, I can install **Parsons** using **pip**

```zsh
(parsons_env) cmarikos@Christinas-MacBook-Air ~ % pip install parsons
```

 
### Step 6: Verify the Installation

To verify that Parsons is installed correctly, I’ll create a simple Python script (`test.py`) that imports **Parsons** and prints a success message.

I opened my parsons_env folder in VS code, and created a new test.py file
```python
from parsons import Table

print('You have successfully installed Parsons!')
```

Run the script:
```zsh
(parsons_env) cmarikos@Christinas-MacBook-Air parsons_env % python test.py
You have successfully installed Parsons!
```

### Conclusion

The installation was relatively straightforward, and I didn’t encounter the errors I faced in previous attempts. The knowledge I gained from earlier experiences, especially regarding the **Python version issue**, was helpful. I also appreciated the clearer instructions in the installation guide this time. Overall, I’m pleased with the installation process and am excited to start using **Parsons** for my work.

