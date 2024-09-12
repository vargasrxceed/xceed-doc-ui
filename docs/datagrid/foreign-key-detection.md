import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Foreign Key Detection

Through the `DataGridCollectionView[Source]` (as well as other collection views that derive from `DataGridCollectionView[Source]Base`), foreign key constraints defined by a **DataTable** or **DataView** used as a data source, as well as enums, can be automatically detected and displayed and edited, through a ComboBox, as the corresponding value rather than its key. In order to automatically detect foreign key constraints, the collection view's `AutoCreateForeignKeyDescriptions` property **must** be set to true, which will result in a `DataGridForeignKeyDescription` being created for each constraint in the `ConstraintCollection` of the **DataTable** or **DataView** used as a source as well as for each enum.

If `AutoCreateForeignKeyDescriptions` is set to **false** or the underlying data source is not a **DataTable** or **DataView**, the foreign key descriptions can be manually specified for each constraint or enum through the ForeignKeyDescription property of the item property that represents the constraint or enum. When manually defining foreign key descriptions because the `AutoCreateForeignKeyDescriptions` property is set to **false**, the **ItemsSource** property and the `ValuePath` properties **must** be set in order to specify the source from which the values will be retrieved as well as their path, which represents the primary key of the foreign item in the **ItemsSource**. 

Foreign key descriptions can also be specified when the constraints are automatically detected in the case where modifications need to be made to the descriptions. In that case, if the **ItemsSource** property has not been set, it will be updated to match the value that was automatically detected.

## Foreign Key Configurations
By default, when a foreign key constraint is detected in a **DataTable** or **DataView** and a foreign key description is automatically created, the key will be replaced by the appropriate value (i.e., object). Each column defines a foreign key configuration (see ForeignKeyConfiguration property), which is represented by the `ForeignKeyConfiguration` class, and through which the visual representation of a foreign key description can be configured. When the grid's `AutoCreateForeignKeyConfigurations` property is set to true, any and all foreign key descriptions will be parsed and configurations will be created for each one. If a configuration has been manually created, it will be used rather than the one generated from its corresponding foreign key description.  

By default, if a `CellContentTemplate` is specified, the same template will be applied to the foreign key configuration (see *EmployeeID* column in Example 1). For simpler configurations, or when a cell-content template is not required, a configuration's `ValuePath` and `DisplayMemberPath` can be used to indicate the path to the value on the source object that corresponds to the "key" and the path to its its visual representation, respectively (see *ShipVia* and *CustomerID* columns in Example 1). 

## Custom Key/Value Mappings
By default, foreign key constraints defined by a **DataTable** or **DataView** as well as enums can be automatically detected; however, through a `ForeignKeyConverter`, custom key/value mappings can also be defined. When providing custom key/value mappings, a foreign key converter must be created by deriving from the `ForeignKeyConverter` class and overriding its `GetKeyFromValue` and `GetValueFromKey` methods in which the value for a specified key and the key for a specified value should be returned (see implementation of `PersonForeignKeyConverter` class in Example 2). This converter can then be provided to the `ForeignKeyConverter` property of either a foreign key description or configuration.

If a ForeignKeyConverter has not been explicitly provided for a `ForeignKeyConfiguration`, the converter from its corresponding `DataGridForeignKeyDescription` will be used.

## Cell Editors
Whether a foreign key constraint or enum is automatically detected or manually provided, the end result is that a **ComboBox** is used to edit the content of the corresponding cell unless a cell editor has been explicitly provided, in which case, the provided cell editor will be used. Cell editors that are created to edit foreign key constraints can access information about their parent column, such as the CellContentTemplate to use, through the `CellEditorContext` attached property. For example:

```xml
<xcdg:Column.CellEditor>
  <!-- ForeignKey CellEditor -->
  <xcdg:CellEditor >
     <xcdg:CellEditor.EditTemplate>
        <DataTemplate>
           <DataTemplate.Resources>
              <xcdg:NullToBooleanConverter x:Key="nullToBooleanConverter" />
           </DataTemplate.Resources>
           
           <ListBox x:Name="fkListBox"
                     xcdg:Cell.IsCellFocusScope="True"
                     ItemTemplate="{Binding RelativeSource={RelativeSource Self},
                                            Path=(xcdg:Cell.CellEditorContext).ParentColumn.CellContentTemplate, Mode=OneWay}"
                     ItemContainerStyle="{Binding RelativeSource={RelativeSource Self},
                                                  Path=(xcdg:Cell.CellEditorContext).ForeignKeyConfiguration.ItemContainerStyle,Mode=OneWay}"
                     ItemsSource="{Binding RelativeSource={RelativeSource Self},
                                           Path=(xcdg:Cell.CellEditorContext).ForeignKeyConfiguration.ItemsSource, Mode=OneWay}"
                     SelectedValuePath="{Binding RelativeSource={RelativeSource Self},
                                                 Path=(xcdg:Cell.CellEditorContext).ForeignKeyConfiguration.ValuePath,Mode=OneWay}"
                     DisplayMemberPath="{Binding RelativeSource={RelativeSource Self},
                                                 Path=(xcdg:Cell.CellEditorContext).ForeignKeyConfiguration.DisplayMemberPath,Mode=OneWay}"
                     SelectedValue="{xcdg:CellEditorBinding}" />
           <!-- Only affect Selector if Template or Style is null -->
           <DataTemplate.Triggers>
              <DataTrigger Binding="{Binding RelativeSource={RelativeSource Self},
                                             Path=(xcdg:Cell.CellEditorContext).ParentColumn.CellContentTemplate,
                                             Converter={StaticResource nullToBooleanConverter},
                                             Mode=OneWay}"
                           Value="True">
                 <Setter TargetName="fkListBox"
                         Property="ItemTemplateSelector"
                         Value="{Binding RelativeSource={RelativeSource Self},
                                         Path=(xcdg:Cell.CellEditorContext).ParentColumn.CellContentTemplateSelector,
                                         Mode=OneWay}" />
              </DataTrigger>
              <DataTrigger Binding="{Binding RelativeSource={RelativeSource Self},
                                             Path=(xcdg:Cell.CellEditorContext).ForeignKeyConfiguration.ItemContainerStyle,
                                             Converter={StaticResource nullToBooleanConverter},
                                             Mode=OneWay}"
                           Value="True">
                 <Setter TargetName="fkListBox"
                         Property="ItemContainerStyleSelector"
                         Value="{Binding RelativeSource={RelativeSource Self},
                                         Path=(xcdg:Cell.CellEditorContext).ForeignKeyConfiguration.ItemContainerStyleSelector,
                                         Mode=OneWay}" />
              </DataTrigger>
           </DataTemplate.Triggers>
        </DataTemplate>
     </xcdg:CellEditor.EditTemplate>
  </xcdg:CellEditor>
</xcdg:Column.CellEditor>
```

If the default editor is used, its ItemTemplate property will be updated to match the value displayed in the cell. For example, if a **CellContentTemplate** that displays *FirstName* and *LastName* rather than *EmployeeID* is used, then the ItemTemplate will also display the value in the same format. 

In the case where a required foreign key configuration is not provided (e.g., forego the use of a data-grid collection view or custom key/value mapping), the default text or numeric editor will be used.

## Grouping and Sorting
When the values of a column that has a foreign key configuration are grouped or sorted, they will be grouped or sorted according to the value displayed in the cell rather than its corresponding key.

:::tip
In order for the values to be sorted correctly, the underlying object must implement **IComparable** or a custom sort comparer must be provided through the `SortComparer` property of the column's corresponding **DataGridItemProperty**.
:::

## Using Foreign Keys with TableflowView
When using a **TableflowView**, because more than one group level is displayed in the `GroupHeaderControl`, a reference point is needed to automatically create a DataTemplate that will bind to the group's value and convert it correctly with a ForeignKeyConverter to allow the use of the CellContentTemplate directly, as with a column. That reference point is the `GroupNavigationButton`. Therefore, if you intend to modify how the `GroupNavigation` buttons or values are displayed in the `GroupHeaderControl`, by changing the implicit **DataTemplate** for their corresponding `Group`, and you wish to display several levels, as in our themes, you must use a GroupNavigationButton as a root visual for the `Group` and its **ParentGroups** in order to take advantage of the ForeignKey feature.

## Examples
All examples in this topic assume that the grid is bound to the Orders table of the Northwind database or a collection of Person objects, unless stated otherwise.

<details>

  <summary>Example 1: Defining foreign key configurations</summary>

  The following example demonstrates how to define foreign key configurations for foreign key descriptions that were automatically created from the constraints extracted from the underlying **DataTable**.

  ```xaml
    <Grid xmlns:xcdg="http://schemas.xceed.com/wpf/xaml/datagrid">
      <Grid.Resources>
          <xcdg:DataGridCollectionViewSource x:Key="cvs_orders"
                                            Source="{Binding Source={x:Static Application.Current}, Path=Orders}"
                                            AutoCreateForeignKeyDescriptions="True"/>
      </Grid.Resources>     
      
      <xcdg:DataGridControl x:Name="OrdersGrid"
                            ItemsSource="{Binding Source={StaticResource cvs_orders}}"
                            AutoCreateForeignKeyConfigurations="True">
          <xcdg:DataGridControl.Columns>
            <xcdg:Column FieldName="EmployeeID"
                          Title="Employee">
                <xcdg:Column.CellContentTemplate>
                  <DataTemplate>
                      <StackPanel Orientation="Horizontal">
                        <TextBlock Text="{Binding FirstName}" />
                        <TextBlock Text=" " />
                        <TextBlock Text="{Binding LastName}" />
                      </StackPanel>
                  </DataTemplate>
                </xcdg:Column.CellContentTemplate>
            </xcdg:Column>
            <xcdg:Column FieldName="CustomerID"
                          Title="Customer">
                <xcdg:Column.ForeignKeyConfiguration>
                  <xcdg:ForeignKeyConfiguration DisplayMemberPath="CompanyName"
                                                ValuePath="CustomerID" />
                </xcdg:Column.ForeignKeyConfiguration>
            </xcdg:Column>
            
            <xcdg:Column FieldName="ShipVia"
                          Title="Shipping Company">
                <xcdg:Column.ForeignKeyConfiguration>
                  <xcdg:ForeignKeyConfiguration DisplayMemberPath="CompanyName"
                                                ValuePath="ShipperID" />
                </xcdg:Column.ForeignKeyConfiguration>
            </xcdg:Column>
          </xcdg:DataGridControl.Columns>
      </xcdg:DataGridControl>
    </Grid>
  ```

</details>

<details>

  <summary>Example 2: Custom key/value mappings</summary>

  The following example demonstrates how to bind the grid directly to a `BindingList<Person>` objects and provide a custom key/value mapping through a ForeignKeyConverter, which will return the appropriate employee first and last names for the provided employee ID.

  ```xaml
    <Grid xmlns:xcdg="http://schemas.xceed.com/wpf/xaml/datagrid">
      <Grid.Resources>
          <xcdg:DataGridCollectionViewSource x:Key="cvs_orders"
                                            Source="{Binding Source={x:Static Application.Current}, Path=Orders}"
                                            AutoCreateForeignKeyDescriptions="True"/>
      </Grid.Resources>     
      
      <xcdg:DataGridControl x:Name="OrdersGrid"
                            ItemsSource="{Binding Source={StaticResource cvs_orders}}"
                            AutoCreateForeignKeyConfigurations="True">
          <xcdg:DataGridControl.Columns>
            <xcdg:Column FieldName="EmployeeID"
                          Title="Employee">
                <xcdg:Column.CellContentTemplate>
                  <DataTemplate>
                      <StackPanel Orientation="Horizontal">
                        <TextBlock Text="{Binding FirstName}" />
                        <TextBlock Text=" " />
                        <TextBlock Text="{Binding LastName}" />
                      </StackPanel>
                  </DataTemplate>
                </xcdg:Column.CellContentTemplate>
            </xcdg:Column>
            <xcdg:Column FieldName="CustomerID"
                          Title="Customer">
                <xcdg:Column.ForeignKeyConfiguration>
                  <xcdg:ForeignKeyConfiguration DisplayMemberPath="CompanyName"
                                                ValuePath="CustomerID" />
                </xcdg:Column.ForeignKeyConfiguration>
            </xcdg:Column>
            
            <xcdg:Column FieldName="ShipVia"
                          Title="Shipping Company">
                <xcdg:Column.ForeignKeyConfiguration>
                  <xcdg:ForeignKeyConfiguration DisplayMemberPath="CompanyName"
                                                ValuePath="ShipperID" />
                </xcdg:Column.ForeignKeyConfiguration>
            </xcdg:Column>
          </xcdg:DataGridControl.Columns>
      </xcdg:DataGridControl>
    </Grid>
  ```
  The following code provides the implementation of the PersonForeignKeyConverter class. 

  <Tabs>
    <TabItem value="csharp" label="C#" default>

      ```csharp
      public class PersonForeignKeyConverter : ForeignKeyConverter
      {
        public override object GetKeyFromValue( object value, ForeignKeyConfiguration configuration )
        {
          PersonBindingList bindingList = configuration.ItemsSource as PersonBindingList;
          if( bindingList != null )
          {
            Person person = value as Person;
            if( person != null )
            {
              return person.PersonID;
            }
          }
          return -1;
        }
        public override object GetValueFromKey( object key, ForeignKeyConfiguration configuration )
        {
          PersonBindingList bindingList = configuration.ItemsSource as PersonBindingList;
          if( bindingList != null )
          {
            try
            {
              int personID = ( int )key;
              foreach( Person person in bindingList )
              {
                if( person.PersonID == personID )
                {
                  return person;
                }
              }
            }
            catch( Exception )
            {
              // key can be null
            }
          }
          return null;
        }
      }
      ```
    </TabItem>
    <TabItem value="vbnet" label="VB.NET">

      ```vbnet
      Public Class PersonForeignKeyConverter
             Inherits ForeignKeyConverter
        Public Overrides Function GetKeyFromValue( value As Object, configuration As ForeignKeyConfiguration ) As Object
          Dim bindingList As PersonBindingList = TryCast( configuration.ItemsSource, PersonBindingList )
          If Not bindingList Is Nothing Then
            Dim person As Person = TryCast( value, Person )
            If Not person Is Nothing Then
              Return person.PersonID
            End If
          End If
          Return -1
        End Function
        Public Overrides Function GetValueFromKey( key As Object, configuration As ForeignKeyConfiguration ) As Object
        Dim bindingList As PersonBindingList = TryCast( configuration.ItemsSource, PersonBindingList )
          If Not bindingList Is Nothing Then
            Try
              Dim personID As Integer = CInt( key )
              Dim person As Person
              For Each person In bindingList
                If person.PersonID = personID Then
                  Return person
                End If
              Next person
            Catch e As Exception
              ' key can be nothing
            End Try
          Return Nothing
        End Function
      End Class
      ```
    </TabItem>    
  </Tabs>

  The following code provides the implementation of the OccupationToStringConverter class.

  <Tabs>
    <TabItem value="csharp" label="C#" default>

      ```csharp
      public class OccupationToStringConverter: IValueConverter
      {
        public object Convert( object value, Type targetType, object parameter, System.Globalization.CultureInfo culture )
        {
          if( value != null && value is Occupation)
          {
            string enumString = value.ToString();
            // Start at 1 to ignore the first capitalizes letter.
            for( int i = 1; i < enumString.Length - 1; i++ )
            {
              if( char.IsUpper( enumString[ i ] ) )
              {
                enumString = enumString.Insert( i, " " );
                i++;
              }
            }
            return enumString;
          }
          return null;
        }
        public object ConvertBack( object value, Type targetType, object parameter, System.Globalization.CultureInfo culture )
        {
          return Binding.DoNothing;
        }
      }
      ```
    </TabItem>
    <TabItem value="vbnet" label="VB.NET">

      ```vbnet
      Public Class OccupationToStringConverter Inherits IValueConverter
        Public Function Convert( value As Object, targetType As Type, parameter As Object,
                                culture As System.Globalization.CultureInfo ) As Object Implements IValueConverter.Convert
          If( Not value Is Nothing ) AndAlso ( TypeOf value Is Occupation ) Then
            Dim enumString As String = value.ToString()
            ' Start at 1 to ignore the first capitalizes letter.
            Dim i as Integer = 1
            For i To i < enumString.Length - 1
              If char.IsUpper( enumString( i ) ) Then
                enumString = enumString.Insert( i, " " )
                i++
              End If
            Next i
            Return enumString
          End If
          Return Nothing
        End Function
        Public Function ConvertBack( value As Value, targetType As Type, parameter As Object,
                                    culture As System.Globalization.CultureInfo ) As Object Implements IValueConverter.ConvertBack
          Return Binding.DoNothing
        End Function
      End Class
      ```
    </TabItem>    
  </Tabs>

</details>