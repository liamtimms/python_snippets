# python_snippets
a place to throw some python code patterns


## Multiproc

```python

import multiprocessing as mp
from functools import partial


def function(x, y, z):
    return

def pool_handler_multifile(x, y, z_list):
    print("starting pool")
    with mp.Pool(mp.cpu_count()) as pool:
        pool.map(
            partial(function, x, y), z_list)
```

## NiFTI ROI extraction

```python
def roi_cut(scan_img, roi_img, t, r):
    """
    Crop part of image using roi, give back crop and values.

    Parameters
    ----------
    scan_img : numpy array
    roi_img : numpy array
    t : string
        string denoting type of crop
    r : value of region of interest from the roi_img

    Returns
    -------
    crop_img : numpy array
    vals_df : pandas DataFrame

    """
    if t == 'equal':
        roi = (roi_img == r).astype(int)
    elif t == 'greater':
        roi = (roi_img >= r).astype(int)
    else:
        print('need valid roi cut type')

    roi = roi.astype('float')
    # zero can be a true value so mask with nan
    roi[roi == 0] = np.nan
    crop_img = np.multiply(scan_img, roi)

    vals = np.reshape(crop_img, -1)
    vals_df = pd.DataFrame()
    vals_df[r] = vals
    vals_df.dropna(inplace=True)
    vals_df.reset_index(drop=True, inplace=True)
    return crop_img, vals_df

def roi_extract(scan_img, roi_img, fname, seg_type, save_dir):
    """
    Extract, save and summarize every region in an roi image.resample_to_img

    Mostly calls roi_cut and df.describe

    Parameters
    ----------
    scan_img : numpy array
    roi_img : numpy array
    seg_type : string
    save_dir : string

    Returns
    -------
    summary_df : pandas DataFrame

    """

    t = 'greater'
    r = -1000
    crop_img, vals_df = roi_cut(scan_img, roi_img, t, r)
    summary_df = vals_df.describe()
    unique_roi = np.unique(roi_img)
    print(unique_roi)

    t = 'equal'
    for r in unique_roi:
        crop_img, vals_df = roi_cut(scan_img, roi_img, t, r)
        r_summary_df = vals_df.describe()
        summary_df = pd.merge(summary_df,
                              r_summary_df,
                              left_index=True,
                              right_index=True)
        vals_df = vals_df.round(3)
        print('Head is :')
        print(vals_df.head())
        print('Tail is :')
        print(vals_df.tail())
        print(r_summary_df.head())
        save_name = (fname + '_DATA_' + 'seg-{}_r-' + str(int(r)) +
                     '.csv').format(seg_type)
        vals_df.to_csv(os.path.join(save_dir, save_name), index=False)
        print('Data saved as:')
        print(os.path.join(save_dir, save_name))
    return summary_df


```

## mkdir -p

```python
if not os.path.exists(save_dir):
    os.makedirs(save_dir)
```
