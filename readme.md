## App Review

## App.tsx

getData should be formatted as follows:

    `const getData = async () => {
        const response = await fetch("./data.json");
        const responseData = await response.json();
        return responseData;
    };`

getData function should be executed in useEffect not outside of it.
getData should be executed as follows in the useEffect hook:

    `useEffect(() => {
        getData()
            .then((data) => {
                setData(data);
            })
            .catch((err) => {
                setLoading(true);
            });
    }, []);`

We also need to remove data from the dependencies in the useEffect hook, as it will prevent the useEffect hook for running an infinite loop

`if (data?.features && !visibleFeatures.length) {
  setVisibleFeatures(data?.features);
}` 

this does not make much sense and is not really readable. We could instead grab the data from the getData response and set the visibleFeatures from there no need to repeat ourselves here. It needs to be implemented as follows:

    `useEffect(() => {
        getData()
            .then((data) => {
                setData(data);
                setVisibleFeatures(data.features);
            })
            .catch((err) => {
                setLoading(true);
            });
    }, []);`

Rendering the code in this manner is neither readable nor effective, 

```
    return data === undefined ? (
    <div>Loading...</div>
    ) : (
    ...rest of content
```

It should be implemented as follows:
    
`    <>
        {!data ? (
            <div>Loading...</div>
        ) : (
            ...rest of content
        )}
        }
    </>`

    An even better approach would be to implement a loading state, we could implement it as follows:
        `const [loading, setLoading] = useState<boolean>(true);`
        `useEffect(() => {
            getData()
                .then((data) => {
                    setData(data);
                    setVisibleFeatures(data.features);
                    setLoading(false);
                })
                .catch((err) => {
                    setLoading(true);
                });
        }, []);`

        Then we could return the content like this:
            `return (
                <>
                    {loading ? (
                        <div>Loading...</div>
                    ) : (
                        ...rest of content
                    )}
                </>
            );`
            
## header.tsx
No issues identified.

## index.tsx
No issues identified.

## map.tsx

export const Map = ({
  ramps,
  visibleFeatures,
  setVisibleFeatures,
}: {
  ramps: RampData;
  visibleFeatures: Feature<MultiPolygon, RampProperties>[] | [];
  setVisibleFeatures: Dispatch<
    SetStateAction<Feature<MultiPolygon, RampProperties>[] | []>
  >;
})

This is not very readable. We could define an interface for map as follows:
    `interface MapProps {
        ramps: RampData;
        visibleFeatures: Feature<MultiPolygon, RampProperties>[] | [];
        setVisibleFeatures: Dispatch<
            SetStateAction<Feature<MultiPolygon, RampProperties>[] | []>
        >;
    }`

We could enhance readibility even more by creating a feature type for the map as follows:
    type visibleFeatures = Feature<MultiPolygon, RampProperties>[] | []

Then implement the interface as follows:
    `interface MapProps {
        ramps: RampData;
        visibleFeatures: visibleFeatures;
        setVisibleFeatures: Dispatch<
            SetStateAction<visibleFeatures>
        >;
    }`
Again one of the most important concepts of software engineering, DRY.
    

Then implement it in the component as follows:
    const Map: React.FC<MapProps> = ({
        ramps,
        visibleFeatures,
        setVisibleFeatures,
    }) => {
        ... rest of content


  This is not needed!
  if (!visibleFeatures) {
    visibleFeatures = ramps.features;
  }
  visibleFeatures is already the same as ramps.features;

  This is not the best way to define an array of elements in react
  const features: JSX.Element[] = [];

  Instead we could define it as follows:
    const features: React.ReactNode[] = [] 

This loop is not the right way to create an effective and readable list map of features
`      for (var i = 0; i < ramps.features.length; i++) {
    var f = ramps.features[i];
    features.push(
      <Marker
        key={f.id}
        latitude={f.geometry.coordinates[0][0][0][1]}
        longitude={f.geometry.coordinates[0][0][0][0]}
        onClick={() => {
          setPopupData(f);
        }}
      />
    );
  }`
    This loop should be implemented as follows:  
    const Features= ramps.features.map((f) => {
        return (
            <Marker
                key={f.id}
                latitude={f.geometry.coordinates[0][0][0][1]}
                longitude={f.geometry.coordinates[0][0][0][0]}
                onClick={() => {
                    setPopupData(f);
                })
            />
        );
    }
    );

    This function should be wrapped with the React.useCallback hook
      const filterVisibleRamps = (evt: ViewStateChangeEvent) => {
            const bounds = evt.target.getBounds();
            const visibleRamps = ramps.features.filter(
            ({ geometry: { coordinates } }) => {
                let inBounds = true;
                console.log(coordinates);
                coordinates[0][0].map((coordKvPair) => {
                if (!bounds.contains(coordKvPair as [number, number])) {
                    inBounds = false;
                }
                });
                return inBounds;
            }
            );
            setVisibleFeatures(visibleRamps);
    };

    Insted of the above the function should be wrapped as follow:
      const filterVisibleRamps = React.useCallback( () => (evt: ViewStateChangeEvent) => {
            const bounds = evt.target.getBounds();
            const visibleRamps = ramps.features.filter(
            ({ geometry: { coordinates } }) => {
                let inBounds = true;
                console.log(coordinates);
                coordinates[0][0].map((coordKvPair) => {
                if (!bounds.contains(coordKvPair as [number, number])) {
                    inBounds = false;
                }
                });
                return inBounds;
            }
            );
            setVisibleFeatures(visibleRamps);
    }, []);


## materials-pie-chart.tsx

The following function:
const calculateData = (ramps: RampData) => {
  const data: ChartDatum[] = [];

  try {
    ramps.features.forEach((feature) => {
      const index = data.findIndex((w) => w.id === feature.properties.material);
      if (index > -1) {
        data[index] = {
          id: feature.properties.material,
          value: data[index].value + 1,
        };
      } else {
        data.push({
          id: feature.properties.material,
          value: 1,
        });
      }
    });
  } catch (ex) {
    console.error(ex);
  }

  return data;
};

Could be wrapped in React.useMemo as it renders a large list of items and it will help with performance.
    const data = React.useMemo(() => calculateData(ramps), [ramps]);

## table.tsx

The following map should include keys as swell as all other maps in this app, Keys help React identify which items have changed, are added, or are removed. Keys should be given to the elements inside the array to give the elements a stable identity.
    Bad
          {data.map((row) => (
            <TableRow>
              <TableCell>{row.rec_id}</TableCell>
              <TableCell>{row.material}</TableCell>
              <TableCell>{row.area_}</TableCell>
              <TableCell>{row.condition}</TableCell>
              <TableCell>{row.owner}</TableCell>
              <TableCell>{row.update_dat}</TableCell>
            </TableRow>
          ))}

    Good
            {data.map((row) => (
                <TableRow key={row.rec_id}>
                    <TableCell>{row.rec_id}</TableCell>
                    <TableCell>{row.material}</TableCell>
                    <TableCell>{row.area_}</TableCell>
                    <TableCell>{row.condition}</TableCell>
                    <TableCell>{row.owner}</TableCell>
                    <TableCell>{row.update_dat}</TableCell>
                </TableRow>
            ))}
