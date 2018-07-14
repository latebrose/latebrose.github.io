# 属性可以用A_B的格式绑定而事件不能
## 属性绑定方法
```CSharp
        // <summary>
        /// Creates data bindings on the view's controls based on the provided properties.
        /// </summary>
        /// <remarks>Parameters include named Elements to search through and the type of view model to determine conventions for. Returns unmatched elements.</remarks>
        public static Func<IEnumerable<FrameworkElement>, Type, IEnumerable<FrameworkElement>> BindProperties = (namedElements, viewModelType) => {

            var unmatchedElements = new List<FrameworkElement>();
#if !XFORMS
            foreach (var element in namedElements) {
                var cleanName = element.Name.Trim('_');
                var parts = cleanName.Split(new[] { '_' }, StringSplitOptions.RemoveEmptyEntries);

                var property = viewModelType.GetPropertyCaseInsensitive(parts[0]);
                var interpretedViewModelType = viewModelType;

                for (int i = 1; i < parts.Length && property != null; i++) {
                    interpretedViewModelType = property.PropertyType;
                    property = interpretedViewModelType.GetPropertyCaseInsensitive(parts[i]);
                }
```
## 事件绑定方法
```CSharp
        /// <summary>
        /// Attaches instances of <see cref="ActionMessage"/> to the view's controls based on the provided methods.
        /// </summary>
        /// <remarks>Parameters include the named elements to search through and the type of view model to determine conventions for. Returns unmatched elements.</remarks>
        public static Func<IEnumerable<FrameworkElement>, Type, IEnumerable<FrameworkElement>> BindActions = (namedElements, viewModelType) => {
            var unmatchedElements = namedElements.ToList();
#if !XFORMS
#if WinRT || XFORMS
            var methods = viewModelType.GetRuntimeMethods();
#else
            var methods = viewModelType.GetMethods();
#endif
            

            foreach (var method in methods) {
                var foundControl = unmatchedElements.FindName(method.Name);
                if (foundControl == null && IsAsyncMethod(method)) {
                    var methodNameWithoutAsyncSuffix = method.Name.Substring(0, method.Name.Length - AsyncSuffix.Length);
                    foundControl = unmatchedElements.FindName(methodNameWithoutAsyncSuffix);
                }
```