### conda activate environment
`conda activate my_env`  
`conda install python=3.6`
### conda install package 
`conda install nltk`
### conda export config file
`conda list -e > requirements.txt`
### init conda shell
`source /root/miniconda3/etc/profile.d/conda.sh`
### create environment
`conda create -n myenv python=3.9`
### list env
`conda env list`
### change environment to flexible
` conda config --set channel_priority flexible`
### install python package with limited resources
`pip install tensorflow -i https://pypi.tuna.tsinghua.edu.cn/simple   --no-cache-dir`

