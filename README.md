
### Template-RTD

#### requirement
sphinx==5.1.1

sphinx-rtd-theme==0.5.2

sphinxcontrib-wavedrom==3.0.1

sphinx-multiversion==0.2.4

recommonmark==0.7.1

sphinx-autobuild

#### compile the doc

   make html     # for html
   
   make latex    # for latex
   
   make latexpdf # for latex (will require latexpdf installed)
   
   make          # list all the available output format

Running on the host local:

sphinx-autobuild source build/html

然后本地 http://127.0.0.1:8000 上即可进行显示


