---
title: CompiledAccessor
lastmod: 2024-04-26T10:23:37-05:00
---
# CompiledAccessor
```csharp
using System.Linq.Expressions;
using System.Reflection;

namespace Inventory.Api.Import
{
    public sealed class CompiledAccessor
    {
        private readonly Dictionary<Type, Dictionary<string, Delegate>> _setterProperties = new();
        private readonly Dictionary<Type, Dictionary<string, Delegate>> _getterProperties = new();

        public CompiledAccessor(Type type)
        {
            Initialize(type);
        }

        public void Initialize(Type type)
        {
            if (type == null)
            {
                throw new ArgumentNullException(nameof(type));
            }

            _setterProperties[type] = new Dictionary<string, Delegate>();
            foreach (var prop in type.GetProperties(BindingFlags.Instance | BindingFlags.Public | BindingFlags.GetProperty | BindingFlags.SetProperty))
            {
                if (prop.CanRead)
                {
                    CreateDelegateRecursive(type, prop, true);
                }

                if (prop.CanWrite)
                {
                    CreateDelegateRecursive(type, prop, false);
                }
            }
        }

        public TValue Get<TObject, TValue>(TObject obj, string propertyName)
        {
            if (obj == null)
            {
                throw new ArgumentNullException(nameof(obj));
            }

            if (string.IsNullOrEmpty(propertyName))
            {
                throw new ArgumentNullException(nameof(propertyName));
            }

            var action = GetActionFor(obj.GetType(), propertyName, true);
            if (action is Func<TObject, TValue> act)
            {
                return act(obj);
            }
            else
            {
                throw new InvalidOperationException($"Property {propertyName} not found.");
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

            var action = GetActionFor(obj.GetType(), propertyName, false);
            if (action is Action<T, object> act)
            {
                act(obj, value);
            }
            else
            {
                throw new InvalidOperationException($"Property {propertyName} not found.");
            }
        }

        private void CreateDelegateRecursive(Type type, PropertyInfo property, bool getter, List<PropertyInfo> parentProperties = null, string prefix = "")
        {
            if (property.PropertyType != typeof(List<Uri>) && property.PropertyType.IsClass && property.PropertyType != typeof(string))
            {
                // For setting first level of subproperties
                foreach (var prop in property.PropertyType.GetProperties(BindingFlags.Instance | BindingFlags.Public | BindingFlags.GetProperty | BindingFlags.SetProperty))
                {
                    if (prop.CanWrite)
                    {
                        CreateDelegateRecursive(
                            type,
                            prop,
                            getter,
                            parentProperties == null
                                ? new List<PropertyInfo> { property }
                                : parentProperties.Append(property).ToList(),
                            prefix + property.Name + ".");
                    }
                }
            }
            else
            {
                if (getter)
                {
                    CreateGetterDelegate(type, property, parentProperties, prefix);
                }
                else
                {
                    CreateSetterDelegate(type, property, parentProperties, prefix);
                }
            }
        }

        private void CreateGetterDelegate(Type type, PropertyInfo property, List<PropertyInfo> parentProperties = null, string prefix = "")
        {
            var paramExpression = Expression.Parameter(type, "it");
            var castExpression = Expression.Convert(paramExpression, type);
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

            var operationExpression = Expression.Convert(propertyExpression, typeof(object));
            var lambdaExpression = Expression.Lambda(typeof(Func<,>).MakeGenericType(type, typeof(object)), operationExpression, paramExpression);
            _getterProperties[type][prefix + property.Name] = lambdaExpression.Compile();
        }

        private void CreateSetterDelegate(Type type, PropertyInfo property, List<PropertyInfo> parentProperties = null, string prefix = "")
        {
            var paramExpression = Expression.Parameter(type, "it");
            var castExpression = Expression.Convert(paramExpression, type);
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
            var lambdaExpression = Expression.Lambda(typeof(Action<,>).MakeGenericType(type, typeof(object)), operationExpression, paramExpression, valueExpression);
            _setterProperties[type][prefix + property.Name] = lambdaExpression.Compile();
        }

        private Delegate GetActionFor(Type type, string propertyName, bool getter)
        {
            if (getter && _getterProperties.TryGetValue(type, out var getterProperties) && getterProperties.TryGetValue(propertyName, out var getterAction))
            {
                return getterAction;
            }

            if (!getter && _setterProperties.TryGetValue(type, out var setterProperties) && setterProperties.TryGetValue(propertyName, out var setterAction))
            {
                return setterAction;
            }

            return null;
        }
    }
}
```