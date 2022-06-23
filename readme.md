## App Review

## Header.tsx

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
    }, [data]);`

`if (data?.features && !visibleFeatures.length) {
  setVisibleFeatures(data?.features);
}` this does not make much sense and is not really readable. We could instead grab the data from the getData response and set the visibleFeatures from there no need to repeat ourselves here. It needs to be implemented as follows:
    `useEffect(() => {
        getData()
            .then((data) => {
                setData(data);
                setVisibleFeatures(data.features);
            })
            .catch((err) => {
                setLoading(true);
            });
    }, [data]);`

Rendering the code in this manner is neither readable nor effective, 
`return data === undefined ? (
<div>Loading...</div>
) : (
    ...rest of content
    `
It should be implemented as follows:
    <>
        {!data ? (
            <div>Loading...</div>
        ) : (
            ...rest of content
        )}
        }
    </>

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
        }, [data]);`

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
