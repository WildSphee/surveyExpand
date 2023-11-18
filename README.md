# surveyExpand
this script is trying to implement a survey result expander, (ethically) 
Useful for when you cannot find enough people to do your survey, you wanna make up results
but cannot to it manually cause you wanna create over 1000 of them. This is the script youre looking for
I didn't write any doc strings cause its a one time thing, but if you got any problems comment in this repo ty much

declaration: its an experiment thing, don't use it for academic frauds kappa

### Start Using

define your range of reference (change_range: Tuple[int1, int2]), the more the merrier
the script will reference row 0-37, and output 125-37 = 88 new rows

```python
df = pd.read_csv("testingResults.csv", header=0)
change_range = (37, 125)
```



# Main functions
### create_series_int
gets column number, expand that column based on the mean and standard deviation

```python
def create_series_int(column_no) -> pd.Series:
    column = pd.to_numeric(df.iloc[:, column_no], errors='coerce')

    # Drop NaN values
    column = column.dropna()

    # Calculate the mean and standard deviation
    mean, std = column.mean(), column.std()
    print(f"{mean=:.2f} {std=:.2f}")
    
    # Generate 100 samples from this distribution
    samples = np.random.normal(mean, std, change_range[1]-change_range[0])

    # If you want to keep the samples within the same range as the original data
    samples = np.clip(samples, column.min(), column.max())

    # Round samples to the nearest integer
    samples = np.round(samples).astype(int)
    return list(samples)
```


### create_ranking
my proudest one, didn't take that long to make but still pretty awesome
the columns should be rankings, like 1st 2nd 3rd 
it will generate a list of ranking with similar results to the original one
Takes in a list of column numbers, the gen_size and pick are variables that control the output variance, play around w them

```python

def create_ranking(columns: list, gen_size: int, pick: int):
    rank_to_num = {"1st": 1, "2nd": 2, "3rd": 3, "4th": 4, "5th": 5}
    num_to_rank = {1: "1st", 2: "2nd", 3: "3rd", 4: "4th", 5: "5th"}
    
    # 14 - 10 = 4
    column_len = columns[-1]+1 -columns[0]
    
    meandict = {}
    stddict = {}
    
    dfcopy = df.copy().iloc[:, columns[0]:columns[-1]+1]
    newdf = pd.DataFrame()
    newdf = dfcopy.replace(rank_to_num)
    # Change column names using df.columns
    newdf.columns = [i for i in range(column_len)]
    
    for i, col in enumerate(columns):
        meandict[i] = newdf[i].mean()
        stddict[i] = newdf[i].std()


    # Or, change column names using df.rename()
    newdf.rename(columns={df.columns[0]: 0, df.columns[1]: 1, df.columns[2]: 2, df.columns[3]: 3}, inplace=True)
    print(f"\nNEWDF:\n{newdf.head()}\n\n")
    
    
    def shuffle_list():
        temp_list = [i+1 for i in range(column_len)].copy()
        # Shuffle the copy
        random.shuffle(temp_list)
        return temp_list
        # Return the shuffled list
    
    data = [shuffle_list() for i in range(gen_size)]
    # Create a DataFrame from the dictionary
    fabdf = pd.DataFrame(data)

    print("MEANDICT\n", meandict)
    def calculate_mean_diff(row):
        collect = []
        for i in range(len(row)):
            collect.append(abs(row[i] - meandict[i]))
        return sum(collect)    
        
    fabdf['sum'] = fabdf.apply(calculate_mean_diff, axis=1)
    fabdf_mean = fabdf['sum'].mean()
    print(f"\nbefore: {fabdf_mean=}")
    fabdf_std = fabdf['sum'].std()
    print(f"before: {fabdf_std=}")
    
    newdf['sum'] = newdf.apply(calculate_mean_diff, axis=1)
    newdf_mean = newdf['sum'].mean()
    newdf_std = newdf['sum'].std()
    print(f"\n{newdf_mean=}")
    print(f"{newdf_std=}")
    
    fabdf['diff'] = fabdf['sum'] - newdf_mean
    # Get the indices of the 10 rows with the largest 'sum' values
    indices_to_drop = fabdf['diff'].nlargest(gen_size - pick).index

    # Drop these rows from the DataFrame
    fabdf = fabdf.drop(indices_to_drop)
    fabdf_mean = fabdf['sum'].mean()
    print(f"\nafter: {fabdf_mean=}")
    fabdf_std = fabdf['sum'].std()
    print(f"after: {fabdf_std=}")
    
    fabdf = fabdf.replace(num_to_rank)
    
    return fabdf.iloc[:,:column_len].reset_index(drop=True)
```


### create_average_value

by average value it can be any value, even strings

```python
def create_average_value(col: int):
    d = df.iloc[:,col].value_counts(normalize=True)
    col = np.random.choice(d.index, size=change_range[1] - change_range[0], p=d.values)
    col = pd.Series(col)
    return col
```
