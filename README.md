# 1. Visibilidad 

Añadir lo siguiente a `RestaurantController` que está en el backend, en la carpeta `controllers`

```JSX
import Sequelize from 'sequelize'
```

Añadir lo siguiente a `RestaurantController` que está en el backend, en la carpeta `controllers`

```JSX
const currentDate = new Date() // Solution (pro)

where: { // Solution (pro)
          visibleUntil: { [Sequelize.Op.or]: [{ [Sequelize.Op.eq]: null }, { [Sequelize.Op.gt]: currentDate }] }
        },
```
Dentro de 
```JSX
const show = async function (req, res) {
  // Only returns PUBLIC information of restaurants
  try {


    const currentDate = new Date() // Solution (pro)


    const restaurant = await Restaurant.findByPk(req.params.restaurantId, {
      attributes: { exclude: ['userId'] },
      include: [{
        model: Product,
        as: 'products',


        where: { // Solution (pro)
          visibleUntil: { [Sequelize.Op.or]: [{ [Sequelize.Op.eq]: null }, { [Sequelize.Op.gt]: currentDate }] }
        },


        include: { model: ProductCategory, as: 'productCategory' }
      },
      {
        model: RestaurantCategory,
        as: 'restaurantCategory'
      }],
      order: [[{ model: Product, as: 'products' }, 'order', 'ASC']]
    }
    )
    // const newProducts = filterNotVisible(restaurant) // Alternative solution
    // restaurant.setProducts(newProducts) // Alternative solution (it is important to use the set)
    res.json(restaurant)
  } catch (err) {
    res.status(500).send(err)
  }
}
```



Añadir lo siguiente a `ProductValidation` que está en el backend, en la carpeta `controllers/validations`

```JSX
check('visibleUntil').optional().isDate().toDate(),
  check('visibleUntil').custom((value, { req }) => {
    const currentDate = new Date()
    if (value && value < currentDate) {
      return Promise.reject(new Error('The visibility must finish after the current date.'))
    } else { return Promise.resolve() }
  }),
  check('availability').custom((value, { req }) => {
    if (value === false && req.body.visibleUntil) {
      return Promise.reject(new Error('Cannot set the availability and visibility at the same time.'))
    } else { return Promise.resolve() }
  })

```
Dentro de 
```JSX
const create = [
  check('name').exists().isString().isLength({ min: 1, max: 255 }).trim(),
  check('description').optional({ checkNull: true, checkFalsy: true }).isString().isLength({ min: 1 }).trim(),
  check('price').exists().isFloat({ min: 0 }).toFloat(),
  check('order').default(null).optional({ nullable: true }).isInt().toInt(),
  check('availability').optional().isBoolean().toBoolean(),
  check('productCategoryId').exists().isInt({ min: 1 }).toInt(),
  check('restaurantId').exists().isInt({ min: 1 }).toInt(),
  check('restaurantId').custom(checkRestaurantExists),
  check('image').custom((value, { req }) => {
    return checkFileIsImage(req, 'image')
  }).withMessage('Please upload an image with format (jpeg, png).'),
  check('image').custom((value, { req }) => {
    return checkFileMaxSize(req, 'image', maxFileSize)
  }).withMessage('Maximum file size of ' + maxFileSize / 1000000 + 'MB'),

  // Solution
  check('visibleUntil').optional().isDate().toDate(),
  check('visibleUntil').custom((value, { req }) => {
    const currentDate = new Date()
    if (value && value < currentDate) {
      return Promise.reject(new Error('The visibility must finish after the current date.'))
    } else { return Promise.resolve() }
  }),
  check('availability').custom((value, { req }) => {
    if (value === false && req.body.visibleUntil) {
      return Promise.reject(new Error('Cannot set the availability and visibility at the same time.'))
    } else { return Promise.resolve() }
  })
]
```

Y añadir tambien lo siguiente a `ProductValidation` que está en el backend, en la carpeta `controllers/validations`

```JSX
check('visibleUntil').optional().isDate().toDate(),
  check('visibleUntil').custom((value, { req }) => {
    const currentDate = new Date()
    if (value && value < currentDate) {
      return Promise.reject(new Error('The visibility must finish after the current date.'))
    } else { return Promise.resolve() }
  }),
  check('availability').custom((value, { req }) => {
    if (value === false && req.body.visibleUntil) {
      return Promise.reject(new Error('Cannot set the availability and visibility at the same time.'))
    } else { return Promise.resolve() }
  })

```
Dentro de 
```JSX
const update = [
  check('name').exists().isString().isLength({ min: 1, max: 255 }),
  check('description').optional({ nullable: true, checkFalsy: true }).isString().isLength({ min: 1 }).trim(),
  check('price').exists().isFloat({ min: 0 }).toFloat(),
  check('order').default(null).optional({ nullable: true }).isInt().toInt(),
  check('availability').optional().isBoolean().toBoolean(),
  check('productCategoryId').exists().isInt({ min: 1 }).toInt(),
  check('restaurantId').not().exists(),
  check('image').custom((value, { req }) => {
    return checkFileIsImage(req, 'image')
  }).withMessage('Please upload an image with format (jpeg, png).'),
  check('image').custom((value, { req }) => {
    return checkFileMaxSize(req, 'image', maxFileSize)
  }).withMessage('Maximum file size of ' + maxFileSize / 1000000 + 'MB'),
  check('restaurantId').not().exists(),
  // Solution
  check('visibleUntil').optional().isDate().toDate(),
  check('visibleUntil').custom((value, { req }) => {
    const currentDate = new Date()
    if (value && value < currentDate) {
      return Promise.reject(new Error('The visibility must finish after the current date.'))
    } else { return Promise.resolve() }
  }),
  check('availability').custom((value, { req }) => {
    if (value === false && req.body.visibleUntil) {
      return Promise.reject(new Error('Cannot set the availability and visibility at the same time.'))
    } else { return Promise.resolve() }
  })
]
```



Añadir lo siguiente a `create-product` que está en el backend, en la carpeta `migrations`

```JSX
visibleUntil: {
        type: Sequelize.DATE
      },

```
Debajo de:
```JSX
order: {
        type: Sequelize.INTEGER
      },
      availability: {
        type: Sequelize.BOOLEAN
      },
```

Añadir lo siguiente a `Product` que está en el backend, en la carpeta `models`

```JSX
isVisible () {
      const currentDate = new Date()
      let isVisible = true

      if (this.visibleUntil && this.visibleUntil < currentDate) {
        isVisible = false
      }

      return isVisible
    }
```

Dentro de 

```JSX
const loadModel = (sequelize, DataTypes) => {
```

Añadir lo siguiente a `Product` que está en el backend, en la carpeta `models`

```JSX
visibleUntil: DataTypes.DATE,
```
Dentro de 
```JSX
Product.init({
    name: DataTypes.STRING,
    description: DataTypes.STRING,
    price: DataTypes.DOUBLE,
    image: DataTypes.STRING,
    order: DataTypes.INTEGER,
    availability: DataTypes.BOOLEAN,

    // Solution
    visibleUntil: DataTypes.DATE,

    restaurantId: DataTypes.INTEGER,
    productCategoryId: DataTypes.INTEGER
  }, {
    sequelize,
    modelName: 'Product'
  })
```





# 2. Frontend


Añadir lo siguiente a `CreateProductScreen` que está en el frontend, en la carpeta `screens/restaurants`

```JSX
visibleUntil: null
const initialProductValues = { name: null, description: null, price: null, order: null, restaurantId: route.params.id, productCategoryId: null, availability: true, visibleUntil: null }
```

Añadir lo siguiente a `CreateProductScreen` que está en el frontend, en la carpeta `screens/restaurants`
```JSX
visibleUntil: yup
      .date()
      .nullable(),
```
Dentro de:
```JSX
const validationSchema = yup.object().shape({
    name: yup
      .string()
      .max(255, 'Name too long')
      .required('Name is required'),
    price: yup
      .number()
      .positive('Please provide a positive price value')
      .required('Price is required'),
    order: yup
      .number()
      .nullable()
      .positive('Please provide a positive order value')
      .integer('Please provide an integer order value'),
    availability: yup
      .boolean(),

    // Solution
    visibleUntil: yup
      .date()
      .nullable(),

    productCategoryId: yup
      .number()
      .positive()
      .integer()
      .required('Product category is required')
  })
```


Añadir lo siguiente a `CreateProductScreen` que está en el frontend, en la carpeta `screens/restaurants`
```JSX
<InputItem
                name='visibleUntil'
                label='Visible until:'
              />
```
Dentro de:
```JSX
<Formik
      validationSchema={validationSchema}
      initialValues={initialProductValues}
      onSubmit={createProduct}>
      {({ handleSubmit, setFieldValue, values }) => (
        <ScrollView>
          <View style={{ alignItems: 'center' }}>
            <View style={{ width: '60%' }}>
              <InputItem
                name='name'
                label='Name:'
              />
              <InputItem
                name='description'
                label='Description:'
              />
              <InputItem
                name='price'
                label='Price:'
              />
              <InputItem
                name='order'
                label='Order/position to be rendered:'
              />

              <DropDownPicker
                open={open}
                value={values.productCategoryId}
                items={productCategories}
                setOpen={setOpen}
                onSelectItem={item => {
                  setFieldValue('productCategoryId', item.value)
                }}
                setItems={setProductCategories}
                placeholder="Select the product category"
                containerStyle={{ height: 40, marginTop: 20, marginBottom: 20 }}
                style={{ backgroundColor: GlobalStyles.brandBackground }}
                dropDownStyle={{ backgroundColor: '#fafafa' }}
              />
              <ErrorMessage name={'productCategoryId'} render={msg => <TextError>{msg}</TextError> }/>





              {/* SOLUTION */}
              <InputItem
                name='visibleUntil'
                label='Visible until:'
              />





              <TextRegular>Is it available?</TextRegular>
              <Switch
                trackColor={{ false: GlobalStyles.brandSecondary, true: GlobalStyles.brandPrimary }}
                thumbColor={values.availability ? GlobalStyles.brandSecondary : '#f4f3f4'}
                // onValueChange={toggleSwitch}
                value={values.availability}
                style={styles.switch}
                onValueChange={value =>
                  setFieldValue('availability', value)
                }
              />
              <ErrorMessage name={'availability'} render={msg => <TextError>{msg}</TextError> }/>

              <Pressable onPress={() =>
                pickImage(
                  async result => {
                    await setFieldValue('image', result)
                  }
                )
              }
                style={styles.imagePicker}
              >
                <TextRegular>Product image: </TextRegular>
                <Image style={styles.image} source={values.image ? { uri: values.image.assets[0].uri } : defaultProductImage} />
              </Pressable>

              {backendErrors &&
                backendErrors.map((error, index) => <TextError key={index}>{error.param}-{error.msg}</TextError>)
              }

              <Pressable
                onPress={ handleSubmit }
                style={({ pressed }) => [
                  {
                    backgroundColor: pressed
                      ? GlobalStyles.brandSuccessTap
                      : GlobalStyles.brandSuccess
                  },
                  styles.button
                ]}>
                <View style={[{ flex: 1, flexDirection: 'row', justifyContent: 'center' }]}>
                  <MaterialCommunityIcons name='content-save' color={'white'} size={20}/>
                  <TextRegular textStyle={styles.text}>
                    Save
                  </TextRegular>
                </View>
              </Pressable>
            </View>
          </View>
        </ScrollView>
      )}
    </Formik>
```





Añadir lo siguiente a `EditProductScreen` que está en el frontend, en la carpeta `screens/restaurants`

```JSX
visibleUntil: null
const initialProductValues = { name: null, description: null, price: null, order: null, restaurantId: route.params.id, productCategoryId: null, availability: true, visibleUntil: null }
```
Añadir lo siguiente a `EditProductScreen` que está en el frontend, en la carpeta `screens/restaurants`
```JSX
visibleUntil: yup
      .date()
      .nullable(),
```
Dentro de:
```JSX
const validationSchema = yup.object().shape({
    name: yup
      .string()
      .max(255, 'Name too long')
      .required('Name is required'),
    price: yup
      .number()
      .positive('Please provide a positive price value')
      .required('Price is required'),
    order: yup
      .number()
      .nullable()
      .positive('Please provide a positive order value')
      .integer('Please provide an integer order value'),
    availability: yup
      .boolean(),

    // Solution
    visibleUntil: yup
      .date()
      .nullable(),

    productCategoryId: yup
      .number()
      .positive()
      .integer()
      .required('Product category is required')
  })
```


Añadir lo siguiente a `EditProductScreen` que está en el frontend, en la carpeta `screens/restaurants`
```JSX
<InputItem
                name='visibleUntil'
                label='Visible until:'
              />
```
Dentro de:
```JSX
<Formik
      validationSchema={validationSchema}
      initialValues={initialProductValues}
      onSubmit={createProduct}>
      {({ handleSubmit, setFieldValue, values }) => (
        <ScrollView>
          <View style={{ alignItems: 'center' }}>
            <View style={{ width: '60%' }}>
              <InputItem
                name='name'
                label='Name:'
              />
              <InputItem
                name='description'
                label='Description:'
              />
              <InputItem
                name='price'
                label='Price:'
              />
              <InputItem
                name='order'
                label='Order/position to be rendered:'
              />

              <DropDownPicker
                open={open}
                value={values.productCategoryId}
                items={productCategories}
                setOpen={setOpen}
                onSelectItem={item => {
                  setFieldValue('productCategoryId', item.value)
                }}
                setItems={setProductCategories}
                placeholder="Select the product category"
                containerStyle={{ height: 40, marginTop: 20, marginBottom: 20 }}
                style={{ backgroundColor: GlobalStyles.brandBackground }}
                dropDownStyle={{ backgroundColor: '#fafafa' }}
              />
              <ErrorMessage name={'productCategoryId'} render={msg => <TextError>{msg}</TextError> }/>





              {/* SOLUTION */}
              <InputItem
                name='visibleUntil'
                label='Visible until:'
              />





              <TextRegular>Is it available?</TextRegular>
              <Switch
                trackColor={{ false: GlobalStyles.brandSecondary, true: GlobalStyles.brandPrimary }}
                thumbColor={values.availability ? GlobalStyles.brandSecondary : '#f4f3f4'}
                // onValueChange={toggleSwitch}
                value={values.availability}
                style={styles.switch}
                onValueChange={value =>
                  setFieldValue('availability', value)
                }
              />
              <ErrorMessage name={'availability'} render={msg => <TextError>{msg}</TextError> }/>

              <Pressable onPress={() =>
                pickImage(
                  async result => {
                    await setFieldValue('image', result)
                  }
                )
              }
                style={styles.imagePicker}
              >
                <TextRegular>Product image: </TextRegular>
                <Image style={styles.image} source={values.image ? { uri: values.image.assets[0].uri } : defaultProductImage} />
              </Pressable>

              {backendErrors &&
                backendErrors.map((error, index) => <TextError key={index}>{error.param}-{error.msg}</TextError>)
              }

              <Pressable
                onPress={ handleSubmit }
                style={({ pressed }) => [
                  {
                    backgroundColor: pressed
                      ? GlobalStyles.brandSuccessTap
                      : GlobalStyles.brandSuccess
                  },
                  styles.button
                ]}>
                <View style={[{ flex: 1, flexDirection: 'row', justifyContent: 'center' }]}>
                  <MaterialCommunityIcons name='content-save' color={'white'} size={20}/>
                  <TextRegular textStyle={styles.text}>
                    Save
                  </TextRegular>
                </View>
              </Pressable>
            </View>
          </View>
        </ScrollView>
      )}
    </Formik>
```














Añadir lo siguiente a `RestaurantDetailScreen` que está en el frontend, en la carpeta `screens/restaurants`

```JSX
const isAboutToBeInvisible = (deadline) => {
    console.log(deadline)
    console.log(typeof (deadline))
    const currentDate = new Date()
    const deadlineDate = new Date(deadline)

    const timeDiff = deadlineDate.getTime() - currentDate.getTime()

    const daysLeft = Math.ceil(timeDiff / (1000 * 3600 * 24))

    return daysLeft <= 7
  }
```
Dentro de:
```JSX
export default function RestaurantDetailScreen ({ navigation, route }) {
```

Añadir lo siguiente a `RestaurantDetailScreen` que está en el frontend, en la carpeta `screens/restaurants`

```JSX
{item.visibleUntil && isAboutToBeInvisible(item.visibleUntil) &&
          <TextRegular textStyle={styles.visible}>Is about to dissapear!</TextRegular>
        }
```
Despues de:
```JSX
<ImageCard
        imageUri={item.image ? { uri: process.env.API_BASE_URL + '/' + item.image } : defaultProductImage}
        title={item.name}
      >
        <TextRegular numberOfLines={2}>{item.description}</TextRegular>
        <TextSemiBold textStyle={styles.price}>{item.price.toFixed(2)}€</TextSemiBold>
        {!item.availability &&
          <TextRegular textStyle={styles.availability}>Not available</TextRegular>
        }
```
Antes de:
```JSX
<View style={styles.actionButtonsContainer}>
          <Pressable
            onPress={() => navigation.navigate('EditProductScreen', { id: item.id })
            }
            style={({ pressed }) => [
              {
                backgroundColor: pressed
                  ? GlobalStyles.brandBlueTap
                  : GlobalStyles.brandBlue
              },
              styles.actionButton
            ]}>
            <View style={[{ flex: 1, flexDirection: 'row', justifyContent: 'center' }]}>
              <MaterialCommunityIcons name='pencil' color={'white'} size={20} />
              <TextRegular textStyle={styles.text}>
                Edit
              </TextRegular>
            </View>
          </Pressable>
```

Añadir lo siguiente a `RestaurantDetailScreen` que está en el frontend, en la carpeta `screens/restaurants`

```JSX
visible: {
    textAlign: 'right',
    marginRight: 5,
    color: GlobalStyles.brandPrimary
  },
```
Dentro de:
```JSX
const styles = StyleSheet.create({
  container: {
    flex: 1
  },
  row: {
    padding: 15,
    marginBottom: 5,
    backgroundColor: GlobalStyles.brandSecondary
  },
  restaurantHeaderContainer: {
    height: 250,
    padding: 20,
    backgroundColor: 'rgba(0,0,0,0.5)',
    flexDirection: 'column',
    alignItems: 'center'
  },
  imageBackground: {
    flex: 1,
    resizeMode: 'cover',
    justifyContent: 'center'
  },
  image: {
    height: 100,
    width: 100,
    margin: 10
  },
  description: {
    color: 'white'
  },
  textTitle: {
    fontSize: 20,
    color: 'white'
  },
  emptyList: {
    textAlign: 'center',
    padding: 50
  },
  button: {
    borderRadius: 8,
    height: 40,
    marginTop: 12,
    padding: 10,
    alignSelf: 'center',
    flexDirection: 'row',
    width: '80%'
  },
  text: {
    fontSize: 16,
    color: 'white',
    alignSelf: 'center',
    marginLeft: 5
  },
  availability: {
    textAlign: 'right',
    marginRight: 5,
    color: GlobalStyles.brandSecondary
  },


  // Solution
  visible: {
    textAlign: 'right',
    marginRight: 5,
    color: GlobalStyles.brandPrimary
  },


  actionButton: {
    borderRadius: 8,
    height: 40,
    marginTop: 12,
    margin: '1%',
    padding: 10,
    alignSelf: 'center',
    flexDirection: 'column',
    width: '50%'
  },
  actionButtonsContainer: {
    flexDirection: 'row',
    bottom: 5,
    position: 'absolute',
    width: '90%'
  }
```
