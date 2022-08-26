# 1. Domain model

```java
@Entity
Book
```

# 2. Develop the Data Transfer Object (Dto)

When the user interface team starts, the service they receive is not ready to use. They create a mock service with an in-memory database. We will divide the user interface development into three major tasks:

* The data transfer object

* The mock service

* The mock user interface

Remember, the mock user interface will be exactly the same as the real user interface. When the service development team is ready to integrate with the user interface team, we will replace the mock service with the real one.

## Data transfer object

The data transfer object acts as a map between the user interface and the data access object tier. The internal database design should not be disclosed to the user interface. Hence, mapping is required with a data transfer object.

```java
public class ProductDto implements Serializable {
  private static final long serialVersionUID = 1L; private Integer id;
  private String name;

  // getters and setters
}
```

## Develop the Mock Service

`Remove` operation

```java
public class ProductInMemoryDB {
 public void remove(Integer id) {
  ProductDto toRemove = null;
  for (ProductDto dto: list)
   if (dto.getId() == id) toRemove = dto;
  if (toRemove != null) list.remove(toRemove);
 }
}
```

```java
public class ProductMockController {
  @RequestMapping(value="/remove/{productid}", method=RequestMethod.POST)
  @ResponseStatus(value = HttpStatus.NO_CONTENT)
  public void remove(@PathVariable("productid") Integer productid){
	ProductInMemoryDB.INSTANCE.remove(productid);
  }
}
```

## Develop the Mock User Interface

`deleteObject` method

```java
function deleteObject(productid) {
    var productForm = {
        id: productid
    };
    delurl = "/mock/remove/" + productid;
    $.ajax({
        url: delurl,
        type: "POST",
        data: productForm,
        dataType: "json",
        success: function(data, textStatus, jqXHR) {
            loadObjects();
        },
        error: function(jqXHR, textStatus, errorThrown) {
            alert("Error Status Delete:" + textStatus);
        }
    });
}
```

# 3. Develop the Resource Tier

The service development will be divided into four major steps:

* Creating the entity

* Creating the data access tier

* Creating the business service tier

* Creating the JSON based REST service, which will be integrated with the user interface.

```java
@Entity @Table(name = "PRODUCT") public class Product {
  private Integer id;
  private String name;
  @Id @GeneratedValue(strategy = GenerationType.AUTO) @Column(name = "ID")
  public Integer getId() {
    return id;
  }
  public void setId(Integer id) {
    this.id = id;
  }
  @Column(name = "NAME", nullable = false, length = 100) public String getName() {
    return name;
  }
  public void setName(String name) {
    this.name = name;
  }
}
```

# 4. Develop the Data Access Tier

### Find all records operation

```java
@RunWith( SpringJUnit4ClassRunner.class )
@ContextConfiguration( locations = { "classpath:context.xml" } )
@TransactionConfiguration( defaultRollback = true )
@Transactional
public class ProductDaoImplTest {
  @Autowired
  private ProductDao dao ;

  @Test
  public void testGetAll() {
	Assert.assertEquals(0L, dao.getAll().size());
  }
}
```

```java
@Repository
@Transactional
public class ProductDaoImpl  implements ProductDao {
  @Autowired
  private SessionFactory sessionFactory ;
  @Override
  public List<Product> getAll() {
	return sessionFactory.getCurrentSession().createQuery("select product from Product product order by product.id desc").list();
  }
}
```

# 5. Develop the Business Service Tier

### Find all records operation

```java
@RunWith( SpringJUnit4ClassRunner.class )
@ContextConfiguration( locations = { "classpath:context.xml" } )
@TransactionConfiguration( defaultRollback = true )
@Transactional
public class ProductServiceImplTest {
	@Autowired
	private ProductService service;

	@Test
	public void testFindAll() {
		Assert.assertEquals(0L, service.findAll().size());
	}
}
```

```java
@Service
@Transactional
public class ProductServiceImpl implements ProductService{
	@Autowired
	private ProductDao productDao;

	@Autowired
	private ProductMapper productMapper;

	@Override
	public List<ProductDto> findAll() {
		List<Product> products = productDao.getAll();
		List<ProductDto> productDtos = new ArrayList<ProductDto>();
		for(Product product : products){
			productDtos.add(productMapper.mapEntityToDto(product));
		}
		return productDtos;
	}
}
```
