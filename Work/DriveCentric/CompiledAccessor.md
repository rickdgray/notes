---
title: CompiledAccessor
lastmod: 2024-04-26T10:23:37-05:00
---
# CompiledSettor
```csharp
using System.Linq.Expressions;
using System.Reflection;

namespace Inventory.Api.Import
{
    public sealed class CompiledSetter
    {
        private readonly Dictionary<Type, Dictionary<string, Delegate>> _properties = new();

        public CompiledSetter(Type type)
        {
            Initialize(type);
        }

        public void Initialize(Type type)
        {
            if (type == null)
            {
                throw new ArgumentNullException(nameof(type));
            }

            _properties[type] = new Dictionary<string, Delegate>();
            foreach (var prop in type.GetProperties(BindingFlags.Instance | BindingFlags.Public | BindingFlags.GetProperty | BindingFlags.SetProperty))
            {
                if (prop.CanWrite)
                {
                    CreateSetterDelegateRecursive(type, prop);
                }
            }
        }

        public void Set<T>(T obj, string propertyName, object value)
        {
            if (obj == null)
            {
                throw new ArgumentNullException(nameof(obj));
            }

            if (string.IsNullOrEmpty(propertyName))
            {
                throw new ArgumentNullException(nameof(propertyName));
            }

            var action = GetActionFor(obj.GetType(), propertyName);
            if (action is Action<T, object> act)
            {
                act(obj, value);
            }
            else
            {
                throw new InvalidOperationException($"Property {propertyName} not found.");
            }
        }

        private void CreateSetterDelegateRecursive(Type type, PropertyInfo property, List<PropertyInfo> parentProperties = null, string prefix = "")
        {
            if (property.PropertyType != typeof(List<Uri>) && property.PropertyType.IsClass && property.PropertyType != typeof(string))
            {
                // For setting first level of subproperties
                foreach (var prop in property.PropertyType.GetProperties(BindingFlags.Instance | BindingFlags.Public | BindingFlags.GetProperty | BindingFlags.SetProperty))
                {
                    if (prop.CanWrite)
                    {
                        CreateSetterDelegateRecursive(
                            type,
                            prop,
                            parentProperties == null
                                ? new List<PropertyInfo> { property }
                                : parentProperties.Append(property).ToList(),
                            prefix + property.Name + ".");
                    }
                }
            }
            else
            {
                CreateSetterDelegate(type, property, parentProperties, prefix);
            }
        }

        private void CreateSetterDelegate(Type type, PropertyInfo property, List<PropertyInfo> parentProperties = null, string prefix = "")
        {
            var parmExpression = Expression.Parameter(type, "it");
            var castExpression = Expression.Convert(parmExpression, type);
            MemberExpression propertyExpression;
            if (parentProperties != null)
            {
                propertyExpression = Expression.Property(castExpression, parentProperties[0].Name);
                foreach (var prop in parentProperties.Skip(1))
                {
                    propertyExpression = Expression.Property(propertyExpression, prop.Name);
                }

                propertyExpression = Expression.Property(propertyExpression, property.Name);
            }
            else
            {
                propertyExpression = Expression.Property(castExpression, property.Name);
            }

            var valueExpression = Expression.Parameter(typeof(object), property.Name);
            var operationExpression = Expression.Assign(propertyExpression, Expression.Convert(valueExpression, property.PropertyType));
            var lambdaExpression = Expression.Lambda(typeof(Action<,>).MakeGenericType(type, typeof(object)), operationExpression, parmExpression, valueExpression);
            _properties[type][prefix + property.Name] = lambdaExpression.Compile();
        }

        private Delegate GetActionFor(Type type, string propertyName)
        {
            if (_properties.TryGetValue(type, out var properties) && properties.TryGetValue(propertyName, out var action))
            {
                return action;
            }

            return null;
        }
    }
}
```
# CompiledSettor
```csharp
using System.Linq.Expressions;
using System.Reflection;

namespace Inventory.Api.Import
{
    public sealed class CompiledSetter
    {
        private readonly Dictionary<Type, Dictionary<string, Delegate>> _properties = new();

        public CompiledSetter(Type type)
        {
            Initialize(type);
        }

        public void Initialize(Type type)
        {
            if (type == null)
            {
                throw new ArgumentNullException(nameof(type));
            }

            _properties[type] = new Dictionary<string, Delegate>();
            foreach (var prop in type.GetProperties(BindingFlags.Instance | BindingFlags.Public | BindingFlags.GetProperty | BindingFlags.SetProperty))
            {
                if (prop.CanWrite)
                {
                    CreateSetterDelegateRecursive(type, prop);
                }
            }
        }

        public void Set<T>(T obj, string propertyName, object value)
        {
            if (obj == null)
            {
                throw new ArgumentNullException(nameof(obj));
            }

            if (string.IsNullOrEmpty(propertyName))
            {
                throw new ArgumentNullException(nameof(propertyName));
            }

            var action = GetActionFor(obj.GetType(), propertyName);
            if (action is Action<T, object> act)
            {
                act(obj, value);
            }
            else
            {
                throw new InvalidOperationException($"Property {propertyName} not found.");
            }
        }

        private void CreateSetterDelegateRecursive(Type type, PropertyInfo property, List<PropertyInfo> parentProperties = null, string prefix = "")
        {
            if (property.PropertyType != typeof(List<Uri>) && property.PropertyType.IsClass && property.PropertyType != typeof(string))
            {
                // For setting first level of subproperties
                foreach (var prop in property.PropertyType.GetProperties(BindingFlags.Instance | BindingFlags.Public | BindingFlags.GetProperty | BindingFlags.SetProperty))
                {
                    if (prop.CanWrite)
                    {
                        CreateSetterDelegateRecursive(
                            type,
                            prop,
                            parentProperties == null
                                ? new List<PropertyInfo> { property }
                                : parentProperties.Append(property).ToList(),
                            prefix + property.Name + ".");
                    }
                }
            }
            else
            {
                CreateSetterDelegate(type, property, parentProperties, prefix);
            }
        }

        private void CreateSetterDelegate(Type type, PropertyInfo property, List<PropertyInfo> parentProperties = null, string prefix = "")
        {
            var parmExpression = Expression.Parameter(type, "it");
            var castExpression = Expression.Convert(parmExpression, type);
            MemberExpression propertyExpression;
            if (parentProperties != null)
            {
                propertyExpression = Expression.Property(castExpression, parentProperties[0].Name);
                foreach (var prop in parentProperties.Skip(1))
                {
                    propertyExpression = Expression.Property(propertyExpression, prop.Name);
                }

                propertyExpression = Expression.Property(propertyExpression, property.Name);
            }
            else
            {
                propertyExpression = Expression.Property(castExpression, property.Name);
            }

            var valueExpression = Expression.Parameter(typeof(object), property.Name);
            var operationExpression = Expression.Assign(propertyExpression, Expression.Convert(valueExpression, property.PropertyType));
            var lambdaExpression = Expression.Lambda(typeof(Action<,>).MakeGenericType(type, typeof(object)), operationExpression, parmExpression, valueExpression);
            _properties[type][prefix + property.Name] = lambdaExpression.Compile();
        }

        private Delegate GetActionFor(Type type, string propertyName)
        {
            if (_properties.TryGetValue(type, out var properties) && properties.TryGetValue(propertyName, out var action))
            {
                return action;
            }

            return null;
        }
    }
}
```