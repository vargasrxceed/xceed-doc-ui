# Removing Data

In addition to the insertion events, the `DataGridCollectionView` and `DataGridCollectionViewSource` classes also expose the RemovingItem and `ItemRemoved` events, which are raised when the `Remove` or `RemoveAt` methods have been called to signal that an item is about to be removed or has been removed, respectively, from the underlying data source.

If the process is manually handled, the Handled property received in the event parameters of the `RemovingItem` event must be set to true. The removal process can also be canceled by setting the Cancel property to true.

## Master/Detail
In addition to the `RemovingItem` and `ItemRemoved` events being raised by the master DataGridCollectionView, child details will also raise these events on the master collection view when items are about to be removed or have been removed from their underlying data source.