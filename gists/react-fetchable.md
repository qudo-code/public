
# React Fetchable
Typed request handler for React that manages its own state.
```tsx
// Fetchable pattern used for all requests
type Fetchable<T> = {
  isFetched: boolean;
  isLoading: boolean;
  data: null | T;
  error: null | string;
};

// Spawn default state for fetchable when needed
const FETCHABLE = () => ({
  isFetched: false,
  isLoading: false,
  data: null,
  error: null,
});

// Hook for typing fetchable
const useFetchable = <T,>() => useState<Fetchable<T>>(FETCHABLE());

// Accept a promise to run and auto set fetchable isLoading and error states.
const handleFetchable = async (
  action: () => Promise<any>,
  setFetchable: (fn: (fetchable: Fetchable<any>) => Fetchable<any>) => void
) => {
  setFetchable((prev: Fetchable<any>) => ({
    ...prev,
    isLoading: true,
  }));

  try {
    const data = await action();

    setFetchable((prev: Fetchable<any>) => ({
      ...prev,
      data,
    }));
  } catch (error) {
    console.error(error);

    setFetchable((prev: Fetchable<any>) => ({
      ...prev,
      error: String(error),
    }));
  } finally {
    setFetchable((prev: Fetchable<any>) => ({
      ...prev,
      isLoading: false,
      isFetched: true,
    }));
  }
};
```
