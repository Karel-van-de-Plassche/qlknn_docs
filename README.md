To have maximum teamwork, let's set up both of our environments on rs. I assume you have SSH keys for GitHub
pip install netcdf4
- Create GKDB user (to pull from my NNDB):
    As in here: https://app.asana.com/0/289633837446126/314421882631984 might need to write a guide at some point, but let's do this together now 
    Put username/password in ~/<200b>.pgpass for convenience:
    *:*:*:username:password

- Install QLKNN tools:
```
git clone git@github.com:Karel-van-de-Plassche/QLKNN-develop.git
cd QLKNN-develop
pip install -e .[all] --user
```

- Install dataslicer (assuming you have accepted my invite):
```
pip install nodejs bokeh==0.12.6 --user
git clone git@github.com:Karel-van-de-Plassche/QuaLiKiz-dataslicer.git
cd QuaLiKiz-dataslicer
git submodule init<200b>
git submodule update
export PATH=$PATH:/Rijnh/Shares/Departments/Fusiefysica/IMT/karel/nodejs/bin
```
- Install optional dependencies to load NetCDF4 huge datafiles:
```
pip install netcdf --user
```

## Usage
Two main analysis tools are the `quickslicer` and the `dataslicer`

### Quickslicer
This is the tools that quickly slices through the 1D slices of the dataset. It written for multicore and is used to determine physics parameters in threshold dimensions, although it can be used to slice through all dimensions. The script is located in `QLKNN-develop/qlknn/plots/slicer.py`. If you are using it manually:
1. Copy file to local disk to prevent some rs filesystem weirdness
``` bash
cp /Rijnh/Shares/Departments/Fusiefysica/IMT/karel/gen3_7D_nions0_flat_filter8.h5 $HOME/QLKNN-develop
```
2. Modify the script to run in debug mode (done automatically for rs*)
``` diff
-         mode = 'quick'
-         submit_to_nndb = True
+         #mode = 'quick'
+         #submit_to_nndb = True
```
3. Turn of autofinding of Networks to slice (done automatically for rs*) 
``` diff
-         slicedim, style, nns = nns_from_NNDB(100, only_dim=7)
-         #slicedim, style, nns = nns_from_manual()
+         #slicedim, style, nns = nns_from_NNDB(100, only_dim=7)
+         slicedim, style, nns = nns_from_manual()
```
4. In the function `nns_from_manual`, append the networks you want to see to `dbnns`, and select the slicedim. Note that the quickslicer assumes all networks have the same amount of outputs! For example, put:
``` python
dbnns.append(Network.get_by_id(12)) #TEM network
dbnns.append(Network.get_by_id(14)) #TEM network
slicedim = 'Ati'
```
5. Optionally don't drop to shell after each plot:
``` diff
            slice_latex += ('\\\\\n' + ' {:.2f} &' * len(index)).format(*index).strip(' &')
-           embed()
+           #embed()
            plt.close(fig)
```
6. Run the slicer!
``` bash
python slicer.py
```
### Dataslicer
1. Choose the networks you want to display and add them to the `nns` list. It is located in the `mega_nn.py` file.
``` python
nns = []
nns.append(Network.get_by_id(423).to_QuaLiKizNN())
nns.append(Network.get_by_id(166).to_QuaLiKizNN())
nns.append(Network.get_by_id(331).to_QuaLiKizNN())
```
2. Choose the plots to display in the `analyse.py` file. For example, the heatfluxes `plot_ef`,the seperate ITG, ETG, and TEM fluxes `plot_sepflux`, and the nns themselves `plot_nn`:
``` python
285 plot_nn = plot_nn and True
286 plot_freq = True
287 plot_ef = True
288 plot_pf = False
289 plot_pinch = False
290 plot_df = False
291 plot_grow = False
292 plot_sepflux = True
```
3. Start the dataslicer server!
``` bash
bokeh serve --port 5010 --allow-websocket-origin=$(hostname).rijnh.nl:5010 analyse.py
```
4. Navigate to the bokeh plot webpage, url: webserver:5010
