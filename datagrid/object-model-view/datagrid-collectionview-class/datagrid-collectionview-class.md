# DataGridCollectionView Class

The `DataGridCollectionView` class allows any collection that implements the IEnumerable interface to be grouped, sorted, or filtered. Unlike the `BindingListCollectionView` class, the `DataGridCollectionView` class will never touch the underlying data source, meaning that the original sorting and filtering present in the data source will not be modified. The underlying data source to which the `DataGridCollectionView` is bound can be retrieved through the `SourceCollection` property.

By default, a DataGridItemProperty will be created for each property in the underlying data source and can be retrieved through the `ItemProperties` property of the `DataGridCollectionView` (you can consider these items to be the columns that will end up in a grid). Each item-property object represents the characteristics of a property for an item in the DataGridCollectionView, including the `DataType`, `Name`, `PropertyDescriptor`, `Title`, and `ValuePath`.

When you set the GroupSortStatResultPropertyName property to a valid `StatFunction.ResultPropertyName` defined in `DataGridCollectionView.StatFunctions`, sorting by the `DataGridItemProperty` having a `StatResultPropertyName` will also sort the group by the stat result of that name.

When you set the `GroupSortStatResultComparer` property, the groups are ordered based on whether GroupSortStatResultPropertyName is also set; the passed IComparer will receive the stat result as values. If you do not pass any comparer, the default sort comparer for the returning type of the stat result will be used.
Multiple `DataGridItemProperty` sorting is also supported. For example, if property A and B have a GroupSortStatResultPropertyName that points to a sum, we are sorted by A then by B. When sum of A is equal, the sum of B will be used to determine the group order.

In order to avoid performance degradation, when modifying a cell value, only the group affected by that cell value modification is reordered, so there should not be any significant performance degradation when using this property.

Automatic creation of item properties can be disabled by setting the `AutoCreateItemProperties` property of the `DataGridCollectionViewSource` to false (by default, true) or by specifying so when creating an instance of the `DataGridCollectionView`.

The `DataGridCollectionView` Contains method determines whether an item exists in the current view, while the `GetItemAt` and `IndexOf` methods can be used to retrieve an item at a specific index or retrieve its index, respectively. The number of items contained in a `DataGridCollectionView` can be determined through the Count property. In the case where no items are displayed in the view, the `Count` property will return 0 and the IsEmpty property will return true.

:::tip
Programmatically, when the `DataGridCollectionView` is bound to a data source, the `ItemProperties.Clear` method must be called prior to adding `DataGridItemProperty` objects to the ItemProperties collection to remove items that were automatically added when the `DataGridCollectionView` was instantiated.
:::

Unbound data can be "appended" to a data item through the use of unbound item properties, which are represented by the `DataGridUnboundItemProperty` class. Unlike unbound columns, which can be used to display non-data related information such as a label or controls that allow some sort of action to be carried out, unbound item properties can be used to provide additional data, such as calculated columns. 

## Grouping, Sorting, and Filtering
Data items contained in the view can be grouped using either the default `PropertyGroupDescription` or the `DataGridGroupDescription` (recommended) and adding them to the `GroupDescriptions` property. The final, runtime groups can be retrieved through the `Groups` property. Data items can also be sorted by adding the standard `SortDescription` structures to the `SortDescriptions` property. In addition to the default type-based sorting, custom sorting can also be applied by providing an `IComparer` to the `SortComparer` property of one or more item properties defined in the `DataGridCollectionView` to which a grid is bound.

## Detail Relations (Hierarchical Master/Detail)
If the data source to which a DataGridCollectionView is bound contains hierarchical detail relations, the content of those relations will be displayed as the details of data items in a grid or in another detail. Relations are represented by DataGridDetailDesciption objects and will automatically be created for:

- every `DataRelation` in a `DataTable` (`DataRelationDetailDescription`)
- data items that implement the `IEnumerable` interface (`EnumerableDetailDescription`)
- data items that implement the `IListSource` interface (`ListSourceDetailDescription`)

Automatic creation of detail descriptions can be disabled by setting the `AutoCreateDetailDescriptions` property of the `DataGridCollectionViewSource` to false (by default, **true**) or by specifying so when creating an instance of the `DataGridCollectionView`.

In order for the details of a data item to be displayed (and created), the parent grid or detail's `AutoCreateDetailConfigurations` property must be set to true (by default, false at the grid level and true for detail configurations).

## Inserting and Editing Data Items
The `DataGridCollectionView` and `DataGridCollectionViewSource` classes expose events (see Table 1 in [Inserting Data](/datagrid/fundamentals/providing-inserting-remove/inserting-data)) that are triggered during key stages of inserting a new item into an underlying data source. These events provide full control over the insertion process and make it possible to insert items into a source that does not implement the IBindingList interface.

The `DataGridCollectionView` also exposes the `EditItem`, `CommitEdit`, and `CancelEdit` methods to allow items in the underlying data source to be edited directly. 

## DataGridCollectionViewSource Class
The `DataGridCollectionViewSource` class represents the XAML proxy of the `DataGridCollectionView` class. The `DataGridCollectionViewSource` class is not a view but rather the XAML representation of the `DataGridCollectionView` class.

Most properties and events exposed by the `DataGridCollectionView` class are also exposed by the `DataGridCollectionViewSource` class.

:::tip
Most of the XAML examples demonstrated throughout this documentation make use of the `DataGridCollectionViewSource` class to provide data to a grid.
:::

## Data Virtualization
The `DataGridVirtualizingCollectionView` class and its XAML proxy, the `DataGridVirtualizingCollectionViewSource` class, provide support for data virtualization. Also known as "virtual mode" or "lazy loading", data virtualization provides smart deferred querying of data, support for asynchronous data fetching, preemptive loading, and query abort notifications to ensure a smooth and seamless experience that leaves applications responsive and prevents needless queries to data servers. 