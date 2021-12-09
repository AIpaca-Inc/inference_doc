# Start The First Inference Job on AIbro

<aside class="success">
Play around the executable Colab tutorial <a href = "https://colab.research.google.com/drive/19sXZ4kbic681zqEsrl_CZfB5cegUwuIB#forceEdit=true&sandboxMode=true&scrollTo=Et8ivBtkckme"> here</a>
</aside>

## Step 1: Install

```python
pip install aibro
```

Install [aibro python library](https://pypi.org/project/aibro/) by pip.

If `OSError: protocol not found` shows up, it is caused by missing `/etc/protocols` file. This command should be able to resolve the error: `sudo apt-get -o Dpkg::Options::="--force-confmiss" install --reinstall netbase`

## Step 2: Prepare a formatted model repo

## Step 3: Create an inference API with one-line code

```python
from aibro import Inference

```
