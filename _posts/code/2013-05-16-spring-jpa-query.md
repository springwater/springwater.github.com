---
layout: post
title: "spring jpa query"
tagline: ""
description: ""
category : code
tags: [java, spring, jpa, query]
---
{% include JB/setup %}
###根据entity值查询
{% highlight java %}
abstract protected List<Predicate> getPredicates(CriteriaBuilder cb, Root<M> root, M m);

public List<M> query(M m) {
	CriteriaBuilder builder = em.getCriteriaBuilder();
	CriteriaQuery<M> cq = builder.createQuery(getEntityClass());
	Root<M> root = cq.from(getEntityClass());
	List<Predicate> predicates = getPredicates(builder, root, m);
	return getPageQuery(builder, cq, root, predicates, null);
}

public Page<M> pageQuery(M m, Pageable pageable) {
	CriteriaBuilder builder = em.getCriteriaBuilder();
	CriteriaQuery<M> cq = builder.createQuery(getEntityClass());
	Root<M> root = cq.from(getEntityClass());
	List<Predicate> predicates = getPredicates(builder, root, m);
	return pageQuery(m, builder, cq, root, predicates, pageable);
}

private Page<M> pageQuery(M m, CriteriaBuilder builder, CriteriaQuery<M> cq, Root<M> root,
		List<Predicate> predicates, Pageable pageable) {
	Long total = getCountQuery(m);
	List<M> content = total > pageable.getOffset() ? getPageQuery(builder, cq, root, predicates, pageable)
			: Collections.<M> emptyList();
	return new PageImpl<M>(content, pageable, total);
}

private List<M> getPageQuery(CriteriaBuilder builder, CriteriaQuery<M> cq, Root<M> root,
		List<Predicate> predicates, Pageable pageable) {
	cq.select(root);
	if (pageable != null && pageable.getSort() != null) {
		List<javax.persistence.criteria.Order> orders = new ArrayList<Order>();
		for (Sort.Order order : pageable.getSort()) {
			orders.add(order.isAscending() ? builder.asc(root.get(order.getProperty())) : builder.desc(root
					.get(order.getProperty())));
		}
		cq.orderBy(orders);
	}
	cq.where(predicates.toArray(new Predicate[0]));
	TypedQuery<M> query = em.createQuery(cq);
	if (pageable != null) {
		query.setFirstResult(pageable.getOffset());
		query.setMaxResults(pageable.getPageSize());
	}
	return query.getResultList();
}

private Long getCountQuery(M m) {
	CriteriaBuilder builder = em.getCriteriaBuilder();
	CriteriaQuery<Long> cq = builder.createQuery(Long.class);
	Root<M> root = cq.from(getEntityClass());
	cq.select(builder.count(root));
	List<Predicate> predicates = getPredicates(builder, root, m);
	cq.where(predicates.toArray(new Predicate[0]));
	TypedQuery<Long> query = em.createQuery(cq);
	return query.getSingleResult();
}
{% endhighlight %}

###实现getPredicates
{% highlight java %}
protected List<Predicate> getPredicates(CriteriaBuilder cb, Root<IcircuitTrade> root, IcircuitTrade m) {
	List<Predicate> predicates = new ArrayList<Predicate>();
	if (m != null) {
		EntityType<IcircuitTrade> m_ = root.getModel();
		if (m.getSid() != null) {
			predicates.add(cb.equal(root.get(m_.getSingularAttribute("sid", String.class)), m.getSid()));
		}
		if (m.getCid() != null) {
			predicates.add(cb.equal(root.get(m_.getSingularAttribute("cid", String.class)), m.getCid()));
		}
		if (m.getSpid() != null) {
			predicates.add(cb.equal(root.get(m_.getSingularAttribute("spid", String.class)), m.getSpid()));
		}
		if (m.getCuser() != null) {
			predicates.add(cb.equal(root.get(m_.getSingularAttribute("cuser", String.class)), m.getCuser()));
		}
		if (m.getStat() > 0) {
			predicates.add(cb.equal(root.get(m_.getSingularAttribute("stat", Integer.class)), m.getStat()));
		}
		if (m.getTname() != null) {
			predicates.add(cb.like(root.get(m_.getSingularAttribute("tname", String.class)), "%" + m.getTname()
					+ "%"));
		}
		if (m.getBegin() != null && m.getEnd() != null && "Date".equals(m.getBegin().getClass().getSimpleName())
				&& "Date".equals(m.getEnd().getClass().getSimpleName())) {
			Path<Date> ctime = root.get(m_.getSingularAttribute("ctime", Date.class));
			predicates.add(cb.between(ctime, (Date) m.getBegin(), (Date) m.getEnd()));
		}
	}
	return predicates;
}
{% endhighlight %}