# git clone ... path to project on your git instance


git clone https://github.com/geoHeil/dagster-asset-demo.git
cd dagster-asset-demo
# mamba replace for conda, more quick
conda install mamba -n base -c conda-forge
make create_environment


cd ASSET_DEMO


# follow the instructions below to set the DAGSTER_HOME
# and perform an editable installation (if you want to toy around with this dummy pipeline)
conda activate dagster-asset-demo
pip install --editable .

make dagit
# explore: Go to http://localhost:3000

# optionally enable:
dagster-daemon run
# to use schedules and backfills