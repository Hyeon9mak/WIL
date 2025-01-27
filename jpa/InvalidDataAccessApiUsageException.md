JPA 는 `@Repository` 어노테이션을 사용하는 클래스에서 특정 exception 을 발생시킬 경우 이것을 잡아내어 변환하는 로직을 갖고 있다.

```java
// org.springframework.orm.jpa.EntityManagerFactoryUtils

/**  
 * Convert the given runtime exception to an appropriate exception from the * {@code org.springframework.dao} hierarchy.  
 * Return null if no translation is appropriate: any other exception may * have resulted from user code, and should not be translated. * <p>The most important cases like object not found or optimistic locking failure * are covered here. For more fine-granular conversion, JpaTransactionManager etc * support sophisticated translation of exceptions via a JpaDialect. * @param ex runtime exception that occurred  
 * @return the corresponding DataAccessException instance,  
 * or {@code null} if the exception should not be translated  
 */@Nullable  
public static DataAccessException convertJpaAccessExceptionIfPossible(RuntimeException ex) {  
    // Following the JPA specification, a persistence provider can also  
    // throw these two exceptions, besides PersistenceException.    if (ex instanceof IllegalStateException) {  
       return new InvalidDataAccessApiUsageException(ex.getMessage(), ex);  
    }  
    if (ex instanceof IllegalArgumentException) {  
       return new InvalidDataAccessApiUsageException(ex.getMessage(), ex);  
    }  
  
    // Check for well-known PersistenceException subclasses.  
    if (ex instanceof EntityNotFoundException entityNotFoundException) {  
       return new JpaObjectRetrievalFailureException(entityNotFoundException);  
    }  
    if (ex instanceof NoResultException) {  
       return new EmptyResultDataAccessException(ex.getMessage(), 1, ex);  
    }  
    if (ex instanceof NonUniqueResultException) {  
       return new IncorrectResultSizeDataAccessException(ex.getMessage(), 1, ex);  
    }  
    if (ex instanceof QueryTimeoutException) {  
       return new org.springframework.dao.QueryTimeoutException(ex.getMessage(), ex);  
    }  
    if (ex instanceof LockTimeoutException) {  
       return new CannotAcquireLockException(ex.getMessage(), ex);  
    }  
    if (ex instanceof PessimisticLockException) {  
       return new PessimisticLockingFailureException(ex.getMessage(), ex);  
    }  
    if (ex instanceof OptimisticLockException optimisticLockException) {  
       return new JpaOptimisticLockingFailureException(optimisticLockException);  
    }  
    if (ex instanceof EntityExistsException) {  
       return new DataIntegrityViolationException(ex.getMessage(), ex);  
    }  
    if (ex instanceof TransactionRequiredException) {  
       return new InvalidDataAccessApiUsageException(ex.getMessage(), ex);  
    }  
  
    // If we have another kind of PersistenceException, throw it.  
    if (ex instanceof PersistenceException) {  
       return new JpaSystemException(ex);  
    }  
  
    // If we get here, we have an exception that resulted from user code,  
    // rather than the persistence provider, so we return null to indicate    // that translation should not occur.    return null;  
}
```

가령 아래와 같이 코드를 작성한 경우에 `IllegalArgumentsException` 이 `InvalidDataAccessApiUsageException` 으로 변환되어 던져진다.

```kotlin
@Repository  
class PushDeviceTokenRepositoryAdaptor(  
  private val pushDeviceTokenEntityJpaRepository: PushDeviceTokenEntityJpaRepository,  
) : PushDeviceTokenRepository {  
  override fun save(pushDeviceToken: PushDeviceToken): PushDeviceToken {  
    val entity = pushDeviceToken.toEntity()  
    val savedEntity = pushDeviceTokenEntityJpaRepository.save(entity)  
    return savedEntity.toDomain()  
  }  
  
  override fun findToDeletePushDeviceTokenById(pushDeviceTokenId: Long): ToDeletePushDeviceToken {  
    val entity = pushDeviceTokenEntityJpaRepository.findByIdOrNull(id = pushDeviceTokenId)  
      ?: throw IllegalArgumentException("PushDeviceToken not found. id: $pushDeviceTokenId")  
    return ToDeletePushDeviceToken(pushDeviceTokenId = entity.pushDeviceTokenId)  
  }  
  
  override fun findToDeletePushDeviceTokenByToken(token: PushToken): ToDeletePushDeviceToken {  
    val entity = pushDeviceTokenEntityJpaRepository.findByIdOrNull(id = pushDeviceTokenId)  
      ?: throw IllegalArgumentException("PushDeviceToken not found. token: $token.value")
    return ToDeletePushDeviceToken(pushDeviceTokenId = entity.pushDeviceTokenId)
  }
}
```

이를 회피하기 위해선 `@Repository` 대신에 `@Component` 를 이용해줘도 된다.